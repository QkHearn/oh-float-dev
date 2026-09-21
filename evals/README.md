# float-kit 对照实验

同一模型写 demo。条件 A 关闭本 skill，条件 B 打开。每条 **新开对话**。基础 10 条 + 高级 2 条。

| 文件 | 用途 |
|------|------|
| [README.md](README.md) | 本说明：流程、prompt、硬断言、门槛 |
| [ab-log.md](ab-log.md) | 人工记录表（推荐边跑边填） |
| [ab-log.csv](ab-log.csv) | 同上，便于 Excel / Numbers |

## 流程

1. 填 [ab-log.md](ab-log.md) 环境行（日期、模型、skill commit、如何关 skill）。
2. **无 skill**：移走或禁用 `.cursor/skills/float-kit`。按下面 12 条各开一次新对话，任务句 + 轨迹段原样粘贴。填「无 skill」表。
3. **有 skill**：恢复 skill。同样 12 条、同样新对话。填「有 skill」表。
4. 算基础 `/10`、高级 `/2`、合计 `/12` 和 Δ。Δ 写成「无 4/10，有 9/10，+5/10」，不要只写「提升 50%」。

不要在同一条对话里先无 skill 再开 skill。AI 自报的工具次数以 Cursor 工具记录为准，自报会漏。

## 每条末尾必接（原样）

```text
写完 demo 后，在回复末尾单独给出「工具与轨迹」：
1）调用了哪些工具、各多少次（按实际工具名，如 Read / Grep / Write / StrReplace / Shell）；
2）读取了哪些文件，每个给出完整路径，不要只写文件名。
不要省略。
```

## Prompt（任务句 + 上面轨迹段）

### 基础

| ID | 任务句 | 硬断言（全过才算通过） |
|----|--------|------------------------|
| P1 | 写一个画中画 demo：用户看视频退出后继续以系统小窗播放。 | `PiPWindow`；`VIDEO_PLAY`；create 后 `startPiP`；不在 `aboutToAppear` start；改现有 `Index.ets`（有工程时），不要新建 `PipDemo.ets` |
| P2 | 写一个通话场景的画中画小窗 demo。 | `VIDEO_CALL`；控件组通话族；有 `startPiP` |
| P3 | 写一个会议场景的画中画 demo。 | `VIDEO_MEETING`；同族 `controlGroups` |
| P4 | 写一个直播画中画 demo。 | `VIDEO_LIVE`；同族 `controlGroups` |
| W1 | 写一个闪控窗 demo：退后台后仍显示应用自己的圆角小窗页面。 | `floatView`；`ROUNDED_RECTANGLE`；`FLOAT_VIEW` 弹窗；`setUIContext`；`start`；`main_pages.json` |
| W2 | 写一个横条样式的闪控窗 demo。 | `HORIZONTAL_BAR`；其余同 W1 |
| B1 | 写一个闪控球 demo：贴边球，图标加标题加内容。 | `EMPHATIC`；title；不弹窗；`startFloatingBall`；`click` |
| B2 | 写一个闪控球：只要标题和内容。 | `NORMAL`；title 必传 |
| B3 | 写一个闪控球：只要一行标题。 | `SIMPLE`；只传 title |
| B4 | 写一个静态图标闪控球，文案不要刷新。 | `STATIC`；title+icon；禁止 `updateFloatingBall` |

### 高级

| ID | 任务句 | 硬断言 |
|----|--------|--------|
| A1 | 写一个闪控窗和闪控球绑定的 demo：点击球展开窗，点击窗缩小回球。 | 两边都 create 且未 start → `floatView.bind` → 再 start 一侧；听 `IN_FLOATING_BALL`；窗弹窗、球不弹窗；不要先 start 再 bind，不要自己 start 两套手动切 |
| A2 | 写一个闪控窗加防窥保护的 demo：持续展示时，窥视要保护敏感内容。 | `dlpAntiPeep`；STARTED 后用闪控窗 `windowId` 做 `setAntiPeepMaskLayer`；不用主窗 / `getLastWindow` id；不要编 `FloatViewController` 防窥 API |

## 轨迹记什么

| 字段 | 记法 |
|------|------|
| Read / Grep / Write / StrReplace / Shell | 次数（自报 + 核对） |
| 读取文件 | 完整路径，分号分隔 |
| 写出路径 | ets / json5 完整路径 |
| 通过 | 硬断言全过 = 1 |
| 四份 crib | 有 skill 时：`stage-layout`、`arkui-syntax`、`ui-style`、`examples` 是否都读了 |
| A1 / A2 指导 | A1 是否读 bind；A2 是否读 `guide-float-view` / `guide-dlp-anti-peep` |

视觉、文案、token **不进**通过/不通过。没读 crib 但硬断言全过：仍算通过，轨迹记 crib=0。

## 门槛

| 套件 | 有 skill | Δ |
|------|----------|---|
| 基础 10 | ≥ 9/10 | 不回退 |
| 高级 2 | 2/2 | — |
| 合计 12 | — | ≥ +2/12 且不回退 |

k=1 只报成功率与 Δ。要稳定性则每条 k=3，加报三次全过的条数。
