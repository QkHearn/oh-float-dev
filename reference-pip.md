# 画中画误用清单

签名以 [docs/js-apis-pipWindow.md](docs/js-apis-pipWindow.md) 为准。本文件不列接口表。

模块：`import { PiPWindow } from '@kit.ArkUI'`（typeNode 路径再加 `typeNode`）  
起始：API 11；typeNode 路径 API 12+。无特殊 `ohos.permission.*`。  
syscap：`SystemCapability.Window.SessionManager`

**做什么**：把画面迁到系统小窗，系统画控制条。  
**不要用来**：自定义页面（闪控窗）、贴边球（闪控球）、子窗 / `TYPE_FLOAT`。

两条创建路径（签名见接口文档）：

| 路径 | 何时用 | create |
|------|--------|--------|
| 页面已有 `XComponent` | 画面已在布局里 | `create(config)`，`componentController` 必须与组件同一个 |
| **typeNode（API12+ 推荐）** | Navigation / 自由节点 / 不必把 XComponent 放进当前布局 | `create(config, contentNode)`，`contentNode` 为 `typeNode.createNode(..., 'XComponent', options)` |

`updateContentNode` **仅** typeNode 路径。

## 调用顺序

`isPiPEnabled` → `create`（只拿控制器）→ 注册回调 → 主窗 **已 show** → `startPiP` → `STARTED` → 更新 → `stopPiP`

`create` / `startPiP` 的 Promise 成功 ≠ 用户看见小窗，以 `stateChange` 的 `STARTED` 为准。

## 易错约束

- `context`：组件内 `getHostContext()` 转为 `UIAbilityContext`
- `controlGroups` 必须与 `templateType` 同族，最多 3 个；`VIDEO_PLAY` 下 101 与 102 互斥
- 用户说法 → 模板：播放 `VIDEO_PLAY`；通话 `VIDEO_CALL`；会议 `VIDEO_MEETING`；直播 `VIDEO_LIVE`。未说则 `VIDEO_PLAY`
- 用 Navigation 管页则必填 `navigationId`，且与 `<Navigation id>` 一致
- 画面黑但框还在：内容节点问题，不是 start 失败

## 错误码（应用侧）

| 码 | 改什么 |
|----|--------|
| 401 | config：缺 context/controller、控件组不匹配/冲突 |
| 801 | 先 `isPiPEnabled` |
| 1300013 | 主窗 show 后再 start；Navigation id 配对 |
| 1300015 | 状态机：仅 STOPPED 可 start |
| 1300034 | 已有闪控窗则先停 |

## 误用清单（查错用）

查错按本清单逐条扫，命中就写 E。

1. `create` 成功就以为小窗出来了，没有 `startPiP`
2. 页面 XComponent 路径：用了一个 controller，create 传了另一个
3. typeNode 路径：`contentNode` 空，或 `updateContentNode` 用在非 typeNode 创建的 controller
4. `context` 不是 `UIAbilityContext` → 401
5. 在 `onCreate`/`aboutToAppear` 里 `startPiP`，主窗尚未 show → 1300013
6. `controlGroups` 用了别的模板的枚举，或同时塞了 101 和 102 → 401
7. 用 Navigation 但没设 `navigationId`
8. 没听 `controlEvent`，控制条按钮点了没反应
9. 已有闪控窗还 startPiP → 1300034
10. 重复 start（用户连点）→ 1300015
11. 画面黑但小窗框/控制条在：内容节点
12. 把 `createSubWindow` 或 `TYPE_FLOAT` 当成 PiP
