## 功能性测试用例

> 覆盖写程序交卷门禁与关键能力落点。
>
> **环境分支（FUNC-001~004 共用，缺一不可）：**
>
> | 环境 | 怎样算过 |
> |------|----------|
> | **有工程**：工作区有可改 Stage 应用（`module.json5` + `hvigorw`） | 改文件 + 语法自检落笔 + 自己跑 `hvigorw`，输出含 `CompileArkTS` 通过 |
> | **无工程**：仅 skill 仓 / 无 `module.json5` / 无 `hvigorw`（CI 常见） | 命中能力 + **写明停手原因** + **落点清单** + **语法必检清单**；**不要求** `CompileArkTS`；禁止假装编过；禁止只问路径无分析 |

语法必检清单（有/无工程都要在输出里出现，对照 `references/engineering/arkui-syntax.md`）：

- 会抛 API：`try/catch` + `BusinessError`
- `getWindowId()`：`!== undefined` 再传，禁止 `as number`
- PiP：只用 `setAutoStartEnabled`，禁止 `setAutoStart`
- XComponent：用 `XComponentOptions` 形态
- 无 deprecated 顶层 `router.pushUrl`

---

### FUNC-001 写程序交卷门禁（画中画）

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

在当前应用工程里实现视频播放画中画（默认 XComponent）。

#### 预期

**分流**：`pip`

**有工程时：**

1. 落点：`PipHost`/`Index` + `Page1` + `AVPlayer.ets` + `main_pages.json`；`create` 后等 `STARTED`
2. 输出含语法必检清单（见文首）
3. 自己跑 `hvigorw assembleHap -p product=default --no-daemon`，输出含 `CompileArkTS` 通过
4. 禁止让用户去 DevEco 代编

**无工程时（CI 默认按此判）：**

1. 命中 `pip`
2. 明确写停手原因：无应用工程 / 无 `hvigorw`，不新建框架仓 `entry/`
3. 列出将改落点（同上）+ 语法必检清单
4. 可附骨架要点；**不要求** `CompileArkTS`；禁止假装编过
5. 可以追问工程路径，但不能只有追问、没有 2~3

---

### FUNC-002 闪控窗写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

实现闪控窗：Index 启动，内容页 `FloatPanel`。

#### 预期

- 命中 `float_view`
- 权限三处齐；`setUIContext` 后 `start`，以 `STARTED` 为准；禁止把启动逻辑写进 FloatPanel
- **有工程**：改文件 + 语法必检 + `CompileArkTS` 通过
- **无工程**：停手原因 + 落点（Index / FloatPanel / 权限）+ 语法必检清单；不要求编译日志

---

### FUNC-003 闪控球写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

在 Index 上做闪控球，模板用强调样式。

#### 预期

- 命中 `floating_ball`，模板 `EMPHATIC`
- 不新建球页面，不 `setUIContext`
- **有工程**：改文件 + 语法必检 + `CompileArkTS` 通过
- **无工程**：停手原因 + 落点（Index / 权限）+ 语法必检清单；不要求编译日志

---

### FUNC-004 绑定写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

球窗绑定：点球开窗、点窗回球。

#### 预期

- 命中 `bind`；两边未 start 再 `bind`
- 不要再抄一份完整 `floating-ball` Index
- **有工程**：改文件 + 语法必检 + `CompileArkTS` 通过
- **无工程**：停手原因 + 落点（Index / FloatPanel / bind）+ 语法必检清单；不要求编译日志

---

### FUNC-005 查错：create 当已显示

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（查错）

#### 输入

贴一段代码：`create` 成功后立刻当小窗已显示；用户反馈看不见窗。

#### 预期

- 读 `misuse-*.md`，标出文件:行
- 改法：等状态回调 `STARTED`（闪控窗勿把 `start()` Promise 返回当完成）
- 能改工作区文件就直接改；无文件可改时写出定位与改法即可

---

### FUNC-006 范围外停手

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（分流）

#### 输入

用子窗口 `createSubWindow` 做悬浮小窗。

#### 预期

- `out_of_scope`，一句分流理由后停手
- 不产生 ets，不计入编译分母
