## 功能性测试用例

> 覆盖写程序交卷门禁（语法 + 编译）与关键能力落点。评测时必须有可改的 Stage 应用工程。

---

### FUNC-001 写程序交卷门禁（语法 + 编译）

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

在当前应用工程里实现视频播放画中画（默认 XComponent）。

#### 预期

**分流**：`pip`

**写程序必检（交卷前全部勾上）**：

1. **语法自检**：对照 `reference/engineering/arkui-syntax.md`「写出来必须能编过」
   - 会抛 API 有 `try/catch` + `BusinessError`
   - `getWindowId()` 先 `!== undefined`，禁止 `as number`
   - PiP 用 `setAutoStartEnabled`，禁止 `setAutoStart`
   - XComponent 用 `XComponentOptions` 形态
2. **自己编译**：在应用工程根跑 `hvigorw assembleHap -p product=default --no-daemon`
3. **交卷证据**：输出中带 `CompileArkTS` 通过；失败则改 ets 再编，禁止让用户去 DevEco 代编

**落点**：`PipHost`/`Index` + `Page1` + `AVPlayer.ets` + `main_pages.json`；`create` 后等 `STARTED` 才算显示。

---

### FUNC-002 闪控窗写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

实现闪控窗：Index 启动，内容页 `FloatPanel`。

#### 预期

- 命中 `float_view`
- 权限三处齐；`setUIContext` 后 `start`，以 `STARTED` 为准
- 完成 FUNC-001 同款语法自检 + `CompileArkTS` 通过
- 禁止把启动逻辑写进 FloatPanel

---

### FUNC-003 闪控球写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

在 Index 上做闪控球，模板用强调样式。

#### 预期

- 命中 `floating_ball`，模板 `EMPHATIC`
- 不新建球页面，不 `setUIContext`
- 完成语法自检 + `CompileArkTS` 通过

---

### FUNC-004 绑定写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

球窗绑定：点球开窗、点窗回球。

#### 预期

- 命中 `bind`；两边未 start 再 `bind`
- 不要再抄一份完整 `floating-ball` Index
- 完成语法自检 + `CompileArkTS` 通过

---

### FUNC-005 查错：create 当已显示

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（查错）

#### 输入

贴一段代码：`create` 成功后立刻当小窗已显示；用户反馈看不见窗。

#### 预期

- 读 `misuse-*.md`，标出文件:行
- 改法：等状态回调 `STARTED`（闪控窗勿把 `start()` Promise 返回当完成）
- 能改工作区文件就直接改

---

### FUNC-006 范围外停手

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（分流）

#### 输入

用子窗口 `createSubWindow` 做悬浮小窗。

#### 预期

- `out_of_scope`，一句分流理由后停手
- 不产生 ets，不计入编译分母
