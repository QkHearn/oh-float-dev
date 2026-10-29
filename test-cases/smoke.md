## 冒烟测试用例

> 验证分流表能否正确命中画中画 / 闪控窗 / 闪控球 / 绑定 / 防窥 / 范围外。

---

### SMOKE-001 画中画（视频播放）

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：smoke（路由）

#### 输入

给视频播放页加画中画，退到后台还能以小窗继续播。

#### 预期

- 命中：`pip`（`PiPWindow`，默认 XComponent，模板 `VIDEO_PLAY`）
- 禁止：`float_view`、`floating_ball`、`TYPE_FLOAT`、`createSubWindow`

---

### SMOKE-002 画中画（会议）

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：smoke（路由）

#### 输入

视频会议场景要进系统画中画小窗。

#### 预期

- 命中：`pip`，模板 `VIDEO_MEETING`
- `controlGroups` 与 `controlEvent` 同族

---

### SMOKE-003 闪控窗

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：smoke（路由）

#### 输入

应用退后台后还要继续展示自定义页面内容的小窗。

#### 预期

- 命中：`float_view`
- 禁止：用 PiP 冒充（不要系统 PiP 控制条）

---

### SMOKE-004 闪控球

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：smoke（路由）

#### 输入

做一个贴边小球，只显示标题和进度，不要自定义 UI。

#### 预期

- 命中：`floating_ball`（默认模板 `EMPHATIC`）
- 禁止：给球新建页面 / `setUIContext`

---

### SMOKE-005 绑定

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：smoke（路由）

#### 输入

点小球展开闪控窗，点窗再缩回球。

#### 预期

- 命中：`bind`（`floatView.bind`）
- 读骨架：float-view Index + bind，不要再抄整份 floating-ball Index

---

### SMOKE-006 闪控窗 + 防窥

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：smoke（路由）

#### 输入

闪控窗一直展示，被窥视时要系统蒙层。

#### 预期

- 命中：`float_view+peep`
- 权限含 `DLP_GET_HIDE_STATUS`；`HIDE` 走 `showSystemMaskLayer`

---

### SMOKE-007 范围外（TYPE_FLOAT）

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：smoke（路由）

#### 输入

用 `TYPE_FLOAT` 做个全局悬浮窗。

#### 预期

- 命中：`out_of_scope`
- 明确说不在本 Skill 范围后停手，不写代码

---

### SMOKE-008 范围外（createSubWindow）

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：smoke（路由）

#### 输入

帮我 `createSubWindow` 做一个子窗口当画中画。

#### 预期

- 命中：`out_of_scope`
- 说明应用辅助窗应走 PiP / floatView / floatingBall，然后停手
