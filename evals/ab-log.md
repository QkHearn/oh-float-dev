# float-kit 对照实验记录

实验说明见 [README.md](README.md)。填之前先写环境。每格只记事实。硬断言全过打 `1`，否则 `0`，失败原因写在「失败项」。

**环境**

| 项 | 值 |
|----|----|
| 日期 |  |
| 模型 |  |
| skill commit |  |
| k（每条重跑次数） | 1 |
| 无 skill 如何关闭 |  |

**轨迹段（每条 prompt 末尾都要贴）**

```text
写完 demo 后，在回复末尾单独给出「工具与轨迹」：
1）调用了哪些工具、各多少次（按实际工具名，如 Read / Grep / Write / StrReplace / Shell）；
2）读取了哪些文件，每个给出完整路径，不要只写文件名。
不要省略。
```

**工具次数**：先抄 AI 自报，再和 Cursor 工具栏核对；不一致以 Cursor 为准。

---

## 无 skill

| ID | 任务 | 通过 0/1 | 失败项 | Read | Grep | Write | StrReplace | Shell | 读取文件（完整路径，分号分隔） | 写出路径 | 备注 |
|----|------|----------|--------|------|------|-------|------------|-------|------------------------------|----------|------|
| P1 | 播放画中画 |  |  |  |  |  |  |  |  |  |  |
| P2 | 通话画中画 |  |  |  |  |  |  |  |  |  |  |
| P3 | 会议画中画 |  |  |  |  |  |  |  |  |  |  |
| P4 | 直播画中画 |  |  |  |  |  |  |  |  |  |  |
| W1 | 圆角闪控窗 |  |  |  |  |  |  |  |  |  |  |
| W2 | 横条闪控窗 |  |  |  |  |  |  |  |  |  |  |
| B1 | EMPHATIC 球 |  |  |  |  |  |  |  |  |  |  |
| B2 | NORMAL 球 |  |  |  |  |  |  |  |  |  |  |
| B3 | SIMPLE 球 |  |  |  |  |  |  |  |  |  |  |
| B4 | STATIC 球 |  |  |  |  |  |  |  |  |  |  |
| A1 | 球窗互切 |  |  |  |  |  |  |  |  |  |  |
| A2 | 闪控窗防窥 |  |  |  |  |  |  |  |  |  |  |

无 skill 小计：基础 __ / 10　　高级 __ / 2　　合计 __ / 12

---

## 有 skill

| ID | 任务 | 通过 0/1 | 失败项 | Read | Grep | Write | StrReplace | Shell | 读取文件（完整路径） | 四份 crib 都读了 0/1 | A1读bind / A2读防窥指导 0/1 | 写出路径 | 备注 |
|----|------|----------|--------|------|------|-------|------------|-------|----------------------|----------------------|------------------------------|----------|------|
| P1 | 播放画中画 |  |  |  |  |  |  |  |  |  | — |  |  |
| P2 | 通话画中画 |  |  |  |  |  |  |  |  |  | — |  |  |
| P3 | 会议画中画 |  |  |  |  |  |  |  |  |  | — |  |  |
| P4 | 直播画中画 |  |  |  |  |  |  |  |  |  | — |  |  |
| W1 | 圆角闪控窗 |  |  |  |  |  |  |  |  |  | — |  |  |
| W2 | 横条闪控窗 |  |  |  |  |  |  |  |  |  | — |  |  |
| B1 | EMPHATIC 球 |  |  |  |  |  |  |  |  |  | — |  |  |
| B2 | NORMAL 球 |  |  |  |  |  |  |  |  |  | — |  |  |
| B3 | SIMPLE 球 |  |  |  |  |  |  |  |  |  | — |  |  |
| B4 | STATIC 球 |  |  |  |  |  |  |  |  |  | — |  |  |
| A1 | 球窗互切 |  |  |  |  |  |  |  |  |  |  |  |  |
| A2 | 闪控窗防窥 |  |  |  |  |  |  |  |  |  |  |  |  |

有 skill 小计：基础 __ / 10　　高级 __ / 2　　合计 __ / 12　　四份 crib __ / 10（P/W/B；A 另计指导）

---

## 汇总（填完两表后算）

| 套件 | 无 skill | 有 skill | Δ（有 − 无） | 门禁 |
|------|----------|----------|--------------|------|
| 基础 10 | __ / 10 | __ / 10 |  | ≥ 9/10（有） |
| 高级 2 | __ / 2 | __ / 2 |  | 2/2（有） |
| 合计 12 | __ / 12 | __ / 12 |  | Δ ≥ +2/12 且不回退 |

Δ 写法示例：无 4/10、有 9/10 → `+5/10`（+50pt）。不要只写「提升 50%」。

**失败项可用缩写：** `enum` 模板错 · `start` 缺 start* · `appear` 在 aboutToAppear start · `path` 无路径 · `perm` 权限弹窗错 · `bind` 绑定时机错 · `peep` 防窥 API/windowId 错 · `scope` 范围外该停没停

---

## 硬断言速查

| ID | 全过才打 1 |
|----|------------|
| P1 | PiPWindow；VIDEO_PLAY；create 后 startPiP；不在 aboutToAppear；带 ets 路径 |
| P2 | VIDEO_CALL；通话族控件组 |
| P3 | VIDEO_MEETING；同族 controlGroups |
| P4 | VIDEO_LIVE；同族 controlGroups |
| W1 | floatView；ROUNDED_RECTANGLE；FLOAT_VIEW 弹窗；setUIContext；start；main_pages.json |
| W2 | HORIZONTAL_BAR；其余同 W1 |
| B1 | EMPHATIC；title；不弹窗；startFloatingBall；click |
| B2 | NORMAL；title |
| B3 | SIMPLE；只传 title |
| B4 | STATIC；title+icon；无 updateFloatingBall |
| A1 | 两边 create 且未 start → bind → 再 start 一侧；IN_FLOATING_BALL；不要两套手动切 |
| A2 | dlpAntiPeep；STARTED 后闪控窗 windowId；不用主窗 id；不编 FloatViewController 防窥 API |
