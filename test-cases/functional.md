## 功能性测试用例（问答）

> 聚焦接口答疑与查错问答。不要求改工程、不要求 `CompileArkTS`、不要求追问路径。

---

### FUNC-001 问答：画中画该用哪套

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（问答）

#### 输入

视频播放页要退后台继续以系统小窗播，该用画中画还是闪控窗？默认怎么做？

#### 预期

- 答：用画中画 / `PiPWindow`（不要用闪控窗冒充）
- 提到默认 XComponent / `VIDEO_PLAY`（或等价表述）即可
- 不要求写代码、不要求工程路径

---

### FUNC-002 问答：闪控窗启动顺序

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（问答）

#### 输入

闪控窗怎么启动才算真正显示？`create` 成功算不算已经出来了？

#### 预期

- 答：`create` 只拿控制器；要 `setUIContext` 后 `start`，以状态回调 `STARTED` 为准
- 说明 `start()` Promise 返回 ≠ 已显示（或等价表述）
- 不要求改文件

---

### FUNC-003 问答：闪控球模板默认值

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（问答）

#### 输入

要做贴边小球，用户没说样式，模板选哪个？能不能给球单独建页面？

#### 预期

- 默认模板：`EMPHATIC`
- 不能给球新建页面 / 不要 `setUIContext`
- 不要求写代码

---

### FUNC-004 问答：球窗绑定要点

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（问答）

#### 输入

点球开闪控窗、点窗回球，绑定时有什么注意点？

#### 预期

- 命中绑定 / `floatView.bind`
- 要点：两边未 `start` 再 `bind`（或等价表述）
- 不要求再抄一整份闪控球 Index

---

### FUNC-005 问答：create 成功但看不见窗

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（问答 / 查错）

#### 输入

代码里 `create` 成功后立刻当小窗已显示，用户反馈看不见窗，原因是什么？

#### 预期

- 指出要等状态回调 `STARTED`（或先 `start` 再等回调）
- 口头说明改法即可，不要求改工程文件

---

### FUNC-006 问答：范围外停手

**skill**：hmos-arkui-kit-window-floatwindow  
**类型**：functional（问答）

#### 输入

用子窗口 `createSubWindow` 做悬浮小窗可以吗？

#### 预期

- 答：不在本 Skill 范围 / `out_of_scope`
- 一句分流理由即可（应用辅助窗走 PiP / floatView / floatingBall）
