# 可抄骨架（本地）

日常**不要 WebFetch 官网**。签名以 [docs/](../docs/README.md) 落盘为准；步骤以本目录 + `docs/guide-*.md` 为准。

先保证三种能力的**页面能加载**：登记 `main_pages.json`、路径和加载 API 一致。骨架按 `SKILL.md` 模板对照改枚举；画中画还要同族 `controlEvent`。画中画播放器落 `ets/model/AVPlayer.ets`。

**写程序只读本文件选路径，再按表读齐骨架**（权限另算）。叠加能力不要只读一份。不要通读本目录。

| 能力 | 读 |
|------|----|
| 共用：三种能力页面如何加载 | [merge.md](merge.md)（球 / 画中画 / 窗）。窗、球再读 [permissions.md](permissions.md) |
| 画中画 | [pip.md](pip.md) → 一镜到底读 [pip-xcomponent.md](pip-xcomponent.md)；迁节点读 [pip-typenode.md](pip-typenode.md) |
| 闪控窗 | [permissions.md](permissions.md) + [float-view.md](float-view.md) |
| 闪控球 | [permissions.md](permissions.md)（只声明+ACL，不弹窗）+ [floating-ball.md](floating-ball.md) |
| 绑定 | 窗+球骨架 + [bind.md](bind.md) |
| 闪控窗+防窥 | 闪控窗骨架 + [peep.md](peep.md) |
| 绑定+防窥 | 窗+球骨架 + [bind.md](bind.md) + [peep.md](peep.md) |
| 画中画 + 球窗（+防窥） | [merge.md](merge.md) 叠加节 + [pip.md](pip.md)→xcomponent + [bind.md](bind.md) +（防窥则）[peep.md](peep.md) |

**必须包含**（抄之前核对）：

1. 能力探测：syscap + `isXxxEnabled`
2. 权限三处：见 permissions.md；仅 `FLOAT_VIEW` 弹窗。PiP 无特殊权限
3. 调用顺序与 `BusinessError` 与该能力骨架一致
4. 状态回调；退出 `stop*` + `off*`
5. 画中画先读 `pip.md`。默认 XComponent：同一 controller + `create(config)` 无第二参，进 PiP 不摘组件。typeNode 不是一镜到底。改 `tpl` 后 `controlGroups` / `controlEvent` 必须同族。宿主 `@Entry` 必须能加载
6. 球↔窗：`floatView.bind`，听 `IN_FLOATING_BALL`
7. 防窥：开关、回调、蒙层见 [peep.md](peep.md)
8. 能编过：会抛 API 要 try/catch；禁止抄 `js-apis` 示例；见 [arkui-syntax.md](../docs/arkui-syntax.md)「写出来必须能编过」
