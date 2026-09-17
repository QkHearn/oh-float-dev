---
name: float-kit
description: >-
  辅助 OpenHarmony 悬浮类需求开发：画中画（PiPWindow）、闪控窗（floatView）、闪控球（floatingBall）。
  覆盖能力分流、接口指南、代码查错（指出文件/行/错在哪）、按需求生成 ArkTS 程序、按评审稿写需求设计。
  Use when the user mentions 画中画、PiP、PiPWindow、闪控窗、floatView、闪控球、floatingBall、
  悬浮小窗、球窗绑定、防窥保护组合，或要为这三类写代码/查误用/写需求。
  不覆盖子窗口 createSubWindow、全局悬浮窗 TYPE_FLOAT、模态窗 TYPE_DIALOG。
---

# 悬浮类需求开发（画中画 / 闪控窗 / 闪控球）

只处理三类应用辅助窗口。用户提到子窗 / `TYPE_FLOAT` / 模态窗时：**明确说不在本 Skill 范围**，给一句分流理由后停手，不要硬套这三类 API。

签名、权限、错误码以 [docs/](docs/README.md) **接口落盘**为准。日常不要 WebFetch。用户要求「同步官网」且抓到正文时，才用官网覆盖 `docs/`。`reference-*.md` 只放误用/权限/绑定要点，不列接口表。骨架：[examples.md](examples.md)。指导缺口见 docs/README「缺口」，不要把 `guide-*.md` 当官网全文。

## 何时用哪种模式

| 用户意图 | 模式 |
|----------|------|
| 这是哪种窗 / 该用哪个模块 | **分流** |
| 有哪些接口、权限、错误码、调用顺序 | **接口** |
| 贴了代码 / 报错 / 小窗出不来 | **查错** |
| 给需求，写程序 / 示例 | **写程序** |
| 写需求、评审稿、规格、用例 | **需求设计** |

一次请求可叠加。**先分流，再动手。** 分流只看本文件表格；不要为分流去读接口全文。

## 工作流

```text
进度:
- [ ] 1. 判定能力：pip | float_view | floating_ball | bind | float_view+peep | out_of_scope
        （只看下方分流表；范围外给一句理由后停手）
- [ ] 2. 按模式读文件（见下表），不要把 docs/ 整目录读完
- [ ] 3. 按模式产出；签名以已读的 docs/ 为准
```

| 模式 | 读什么 |
|------|--------|
| **分流** | 本文件分流表。拿不准先问一句，不要 Read `docs/` |
| **接口** | 对应 `js-apis-*.md` 的导入 / create·start / 权限 / 错误码**章节**（Grep 标题后 Read 该段，不要整文件） |
| **查错** | 对应 `reference-*.md`；对上具体码再读 `errorcode-window-float.md` 该条目 |
| **写程序** | `examples.md` 对应骨架；签名对不上再读 `js-apis` 该节。绑定读 floatView bind；防窥读 `guide-*.md` |
| **需求设计** | [req-template.md](req-template.md)；写接口章时再读 `js-apis` 对应节 |

叠加请求：先分流，再按涉及的模式补读，仍不要通读全部落盘。

### 1. 分流（必做）

本步**不要** Read 接口文档。按用户目标选**一种**（绑定标 `bind`；闪控窗+防窥标 `float_view+peep`）：

| 判定 | 能力 | 模块 |
|------|------|------|
| 视频播放 / 通话 / 会议 / 直播；画面迁到系统小窗；系统画控制条 | **画中画** | `PiPWindow`（API12+ 优先 typeNode；页面已有 XComponent 用旧路径） |
| 退后台后继续展示**自定义页面内容**；系统画窗框；应用 `setUIContext` 加载自己的页 | **闪控窗** | `floatView` |
| 贴边小球；只展示标题/内容/图标；**不能自定义 UI** | **闪控球** | `floatingBall` |
| 点击球展开窗、点窗缩小回球 | **绑定** | `floatView.bind` |
| 闪控窗持续展示 + 窥视时保护敏感内容 | **闪控窗 + 防窥** | `floatView` + `dlpAntiPeep` |
| `createSubWindow` / 独立子窗 | **范围外** | 子窗口 |
| `WindowType.TYPE_FLOAT` / 应用自己画整窗 UI | **范围外** | 全局悬浮窗 |
| 视频小窗但用户要完全自定义窗体、不要 PiP 控制条 | **不要用 PiP**；若要自定义页用闪控窗，不要用子窗冒充 |

拿不准时先问一句场景（画面来源？哪种样式？能否自定义 UI？要不要贴边球？要不要防窥？），不要同时生成三套代码。

**模板对照（用户说某一种 → 用对应枚举，不要一律默认第一种）**

画中画 `PiPTemplateType`（`controlGroups` 必须同族，否则 401）：

| 用户说法 | 模板 |
|----------|------|
| 播放、点播、看视频 | `VIDEO_PLAY` |
| 通话、语音/视频电话 | `VIDEO_CALL` |
| 会议、开会 | `VIDEO_MEETING` |
| 直播 | `VIDEO_LIVE` |

闪控球 `FloatingBallTemplate`：

| 用户说法 | 模板 | 必传 |
|----------|------|------|
| 静态、图标+标题、不刷新文案 | `STATIC` | title + icon，禁止 update |
| 标题+内容 | `NORMAL` | title |
| 强调、图标+标题+内容 | `EMPHATIC` | title |
| 只要一行标题 | `SIMPLE` | title |

闪控窗 `FloatViewTemplateType`（系统只这两种窗框；页内容是应用自己的）：

| 用户说法 | 模板 |
|----------|------|
| 圆角小窗、方窗、面板 | `ROUNDED_RECTANGLE` |
| 横条、细条、底部条 | `HORIZONTAL_BAR` |

未说样式时：PiP=`VIDEO_PLAY`，球=`EMPHATIC`，窗=`ROUNDED_RECTANGLE`。控件组细节见对应 `js-apis` 的 Template 节。

**互斥（写程序/查错必须检查）：**

- 已启动闪控窗 → 不能再 `startPiP`（`1300034`）
- 已启动画中画或未绑定的闪控球 → 不能再 `floatView.start`（`1300034`）
- 已启动闪控窗 → 不能再 `startFloatingBall`（`1300034`）
- 绑定成功后：启停任一控制器会同时启停另一侧；同一时刻只展示其中一个

### 2. 三条铁律

1. **`create` 只拿控制器，不建窗。** 用户看得见的窗/球要等对应 `start*`，且以状态回调 `STARTED` 为准（闪控窗文档写明：`start()` Promise 返回 ≠ 启动完成）。
2. **主窗口必须已显示/前台。** PiP/闪控球：主窗已 show；闪控窗：主窗在 foreground。Ability `onCreate` 里直接 start 会失败。
3. **先能力探测再调用。** `canIUse('SystemCapability.Window.SessionManager')` + `isPiPEnabled` / `isFloatViewEnabled` / `isFloatingBallEnabled`。不支持走 `801`，不要假装成功。防窥另加 `canIUse('SystemCapability.Security.DlpAntiPeep')`。

### 3. 接口模式

按能力输出一张表，不要贴整份文档：

- 导入与起始 API 等级
- 权限（`module.json5` 声明 + ACL + 仅 user_grant 运行时弹窗）
- 调用顺序
- 关键参数/模板约束
- 常见错误码 → 应用侧该改什么

先 Grep/Read 该能力 `docs/js-apis-*.md` 的导入、create/start、权限、错误码节，**不要通读全文**。签名以读到的章节为准。reference 只用来核对易错点。

### 4. 查错模式（必须指出错误在哪里）

对照对应 `reference-*.md` 的「误用清单」逐条扫用户代码。错误码含义对不上时再 Read `docs/errorcode-window-float.md` 对应条目。输出**按出现顺序列表**，每条必须含：

```markdown
### E<n> [<能力>] <一句话问题>
- **位置**：`文件路径:行号` 或「函数 `foo` 内、第 N 段代码」（用户没给路径就引用原代码片段）
- **现状**：现在写成了什么
- **错因**：违反哪条规则（接口/权限/生命周期/互斥/模板）
- **表现**：对应错误码或日志特征
- **改法**：给出可粘贴的正确写法（最小补丁，不要整文件重写，除非用户要求）
```

严重级别：

- 🔴 必改：会 401/201/801/13000xx 失败或窗根本出不来
- 🟡 应改：能跑但生命周期/还原/绑定错误
- 🟢 建议：风格、漏听回调、漏 `off`

**禁止**：只说「用法不对」而不标位置；把框架缺陷说成应用误用（查错默认当应用侧误用，除非代码完全正确且日志指向 WM/SceneBoard）。

跨能力先对文末**反模式**；能力细节按对应 `reference-*.md`「误用清单」逐条扫，命中就写 E。

短例（用户代码在 `aboutToAppear` 里调了 `startPiP`）：

```markdown
### E1 [pip] 主窗未 show 就 startPiP
- **位置**：`pages/Index.ets:aboutToAppear`
- **现状**：`aboutToAppear` 里直接 `this.pip.startPiP()`
- **错因**：主窗尚未 show
- **表现**：1300013
- **改法**：改到主窗已显示后的按钮/`onShown`，并以 `stateChange` 的 STARTED 为准
```

### 5. 写程序模式

给需求后生成 **Stage 模型 ArkTS**，对齐 [examples.md](examples.md)。**先按「模板对照」选枚举**，再抄骨架（只改 `templateType` / `template` / 球的必传字段 / PiP 的 `controlGroups`）。PiP：API12+ 默认 typeNode 骨架，除非用户页面已有 XComponent。签名对不上再读 `js-apis` 该节。绑定再读 floatView bind；防窥再读 `guide-*.md`。

程序必须包含：

1. 能力探测（syscap + `isXxxEnabled`）
2. 权限三处都写齐：`module.json5` 声明、签名 Profile 的 ACL、仅 `FLOAT_VIEW` 再 `requestPermissionsFromUser`。PiP 无特殊权限。`USE_FLOAT_BALL` / `DLP_GET_HIDE_STATUS` 为 system_grant，不弹窗。
3. 正确调用顺序与 `BusinessError` 处理
4. 状态回调（PiP `on('stateChange')`；闪控窗 `onStateChange`；闪控球 `on('stateChange'|'click')`）
5. 退出路径：`stop*` + `off*`
6. 若需求涉及球↔窗切换：用 `floatView.bind`，听窗状态 `IN_FLOATING_BALL`；**不要**自己 start 两套再手动切
7. 需求写「闪控窗防窥」：按 [docs/guide-float-view.md](docs/guide-float-view.md) 组合节 + [docs/guide-dlp-anti-peep.md](docs/guide-dlp-anti-peep.md) 写 `dlpAntiPeep`。**不要**在 `FloatViewController` 上编防窥 API。蒙层 windowId 必须是闪控窗 `getWindowProperties().windowId`。

默认只给**一个** Ability 页内的完整可编译片段 + 权限 JSON；用户没要工程脚手架就不要铺满多文件工程。

### 6. 需求设计模式

Read [req-template.md](req-template.md)，按其六章模板输出。本步不必通读接口文档；写「接口章」时再读 `js-apis` 对应节。不要引用或依赖其他 skill。

## 调用顺序速查

**画中画**

```text
isPiPEnabled → create(config[, typeNode]) → on('stateChange'|'controlEvent')
  → （主窗已 show）startPiP → STARTED
  → updateContentSize / updatePiPControlStatus / setPiPControlEnabled
  → stopPiP → STOPPED → off
```

**闪控窗**

```text
isFloatViewEnabled → create(config) → setUIContext(path) 或 setUIContextByName
  → onStateChange → （主窗前台）start → 回调 STARTED 才算成功
  → setWindowSize / switchTemplate
  → stop → offStateChange
```

**闪控球**

```text
isFloatingBallEnabled → create({context}) → on('stateChange'|'click')
  → （主窗已 show）startFloatingBall(params) → STARTED
  → updateFloatingBall（同 template；STATIC 禁止）
  → stopFloatingBall → off
```

**绑定**

```text
两边都 create 且都未 start → floatView.bind(fvCtrl, fbCtrl, fbParams)
  → 再 start() 或 startFloatingBall()（会同时建两个窗，只展示先 start 的那个）
  → stop 任一即两边一起停
```

**闪控窗 + 防窥**

```text
闪控窗 STARTED → canIUse DlpAntiPeep → isDlpAntiPeepSwitchOn
  → 未开：requestAntiPeepOptions
  → on('dlpAntiPeep')
  → HIDE：页面脱敏 + setAntiPeepMaskLayer(闪控窗 windowId)
  → PASS：恢复；STOPPED / 销毁：off
```

## 反模式

- 用 `window.createWindow({type: TYPE_FLOAT})` 实现「闪控」
- 用 `createSubWindow` 冒充画中画
- 把 `PiPWindow.create` 成功当成小窗已显示
- 闪控窗 `start()` then 里立刻 `getWindowProperties` 当已显示
- 同时 new 一套 PiP 和一套 FloatView 当「两种形态」
- 在 `aboutToAppear` 里 start，主窗还没 show
- 闪控球当自定义画布（它没有 `setUIContext`）
- 先 start 再 `floatView.bind`，或未 bind 同时 start 窗和球
- 编造 `FloatViewController` 防窥 API
- `setAntiPeepMaskLayer` 传入主窗 / `getLastWindow` 的 id
