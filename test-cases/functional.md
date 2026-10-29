## 功能性测试用例

> 覆盖写程序交卷门禁与关键能力落点。
>
> **环境分支（FUNC-001~004 共用）：**
>
> | 环境 | 怎样算过 |
> |------|----------|
> | **有工程**：工作区有可改 Stage 应用（`module.json5` + `hvigorw`） | 命中能力 + 改文件；有条件再自检/编译（不硬卡四项清单） |
> | **无工程**：仅 skill 仓 / 无 `module.json5` / 无 `hvigorw`（CI 常见） | **命中能力即可**；追问工程路径也算过；**不要求**停手原因长文、落点清单、语法必检清单、`CompileArkTS` |

---

### FUNC-001 写程序交卷门禁（画中画）

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

在当前应用工程里实现视频播放画中画（默认 XComponent）。

#### 预期

**分流**：`pip`（或明确说画中画 / PiPWindow）

**有工程时：**

1. 改落点相关文件（`PipHost`/`Index`、`Page1`、`AVPlayer.ets`、`main_pages.json` 等，不必一次齐）
2. 有 `hvigorw` 则尽量自编；编不过再改，不硬卡本次必过

**无工程时（CI 默认按此判）：**

任一即可 **PASS**：

1. 判定为 `pip` / 画中画 / PiPWindow；或
2. 说明当前无应用工程并追问工程路径（**只问路径也算过**）

**不要求**：落点清单、语法必检清单、停手原因长文、`CompileArkTS`。  
**FAIL 仅当**：当成别的能力（如闪控窗/球）写完，或假装已编译通过。

---

### FUNC-002 闪控窗写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

实现闪控窗：Index 启动，内容页 `FloatPanel`。

#### 预期

- 命中 `float_view`（或明确说闪控窗 / floatView）
- **有工程**：改 Index / FloatPanel / 权限相关即可
- **无工程**：命中能力或追问工程路径即可；不要求清单与编译

---

### FUNC-003 闪控球写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

在 Index 上做闪控球，模板用强调样式。

#### 预期

- 命中 `floating_ball`（或明确说闪控球）；模板倾向 `EMPHATIC`
- **有工程**：改 Index / 权限即可；不新建球页面
- **无工程**：命中能力或追问工程路径即可；不要求清单与编译

---

### FUNC-004 绑定写程序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（写程序）

#### 输入

球窗绑定：点球开窗、点窗回球。

#### 预期

- 命中 `bind`（或明确说球窗绑定）
- **有工程**：改 Index / FloatPanel / bind 相关即可
- **无工程**：命中能力或追问工程路径即可；不要求清单与编译

---

### FUNC-005 查错：create 当已显示

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（查错）

#### 输入

贴一段代码：`create` 成功后立刻当小窗已显示；用户反馈看不见窗。

#### 预期

- 指出要等状态回调 `STARTED`（或等价表述）
- 能改工作区文件就直接改；无文件可改时写出定位与改法即可

---

### FUNC-006 范围外停手

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（分流）

#### 输入

用子窗口 `createSubWindow` 做悬浮小窗。

#### 预期

- `out_of_scope`（或不在本 Skill 范围），一句理由后停手即可
- 不产生 ets，不计入编译分母
