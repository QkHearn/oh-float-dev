# Skill 内资料（官网落盘）

**以本目录接口落盘为日常契约。** 写程序 / 查错 / 答接口时 Read 本目录对应章节，不要只靠 `reference-*.md`，也不要把本仓 `docs/zh-cn` / DTS 当权威。官网现行正文只在用户要求刷新且抓到完整正文时覆盖本目录。

冲突：刷新成功的官网正文 > 本目录落盘 > `reference-*.md`。

## 刷新

仅当用户说「同步官网 / 刷新 skill」：WebFetch 下表 URL，用现行正文覆盖对应文件。官网页若是空壳（只有「文档中心」），保持本目录不动并说明未覆盖成功。

接口文档初次落盘可能来自与官网同构的 OpenHarmony 文档。刷新一律以官网覆盖。

## 缺口（不要把摘要当官网全文）

| 能力 | 已落盘 | 未落盘 / 非全文 |
|------|--------|-----------------|
| 画中画 | [js-apis-pipWindow.md](js-apis-pipWindow.md) 接口全文 | 官网「画中画开发指导」未进本目录。已知页：https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/pipwindow-typenode 、https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/pipwindow-xcomponent 、https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/window-pipwindow 。写程序以接口 + `examples.md` 为准 |
| 闪控球 | [js-apis-floatingBall.md](js-apis-floatingBall.md) 接口全文 | 无独立 `guide-*.md`。开发步骤以接口文档示例 + `examples.md` 为准 |
| 闪控窗 | 接口全文 + [guide-float-view.md](guide-float-view.md) | `guide-float-view.md` 是要点摘要，不是官网指导全文。官网：https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/float-view-guide |
| 防窥 | [guide-dlp-anti-peep.md](guide-dlp-anti-peep.md) | 同样是要点摘要。官网：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/devicesecurity-dlpantipeep |
| Stage 工程 / ArkUI 语法 | [stage-layout.md](stage-layout.md)、[arkui-syntax.md](arkui-syntax.md) | 摘要，不是入门全文。装饰器专章不落盘 |
| 页面视觉 | [ui-style.md](ui-style.md) | 系统分层参数摘要，不是 UX 规范全文 |

用户要求「同步官网」且抓到完整指导正文时，再覆盖或新增 `guide-*.md`。

## 按能力读

| 能力 | 读这些 |
|------|--------|
| 选型 | [window-type-overview.md](window-type-overview.md) |
| 画中画 | [js-apis-pipWindow.md](js-apis-pipWindow.md) + [../reference-pip.md](../reference-pip.md) |
| 闪控窗 | [js-apis-floatView.md](js-apis-floatView.md) + [guide-float-view.md](guide-float-view.md) + [../reference-float-view.md](../reference-float-view.md) |
| 闪控球 | [js-apis-floatingBall.md](js-apis-floatingBall.md) + [../reference-floating-ball.md](../reference-floating-ball.md) |
| 绑定 | 闪控窗接口 bind/unbind + 指导「球窗绑定」 |
| 防窥组合 | [guide-float-view.md](guide-float-view.md) 组合节 + [guide-dlp-anti-peep.md](guide-dlp-anti-peep.md) |
| 错误码 | [errorcode-window-float.md](errorcode-window-float.md) |
| 权限 | [restricted-permissions-float.md](restricted-permissions-float.md) + [declare-permissions.md](declare-permissions.md) + [declare-permissions-in-acl.md](declare-permissions-in-acl.md) |
| 写程序·工程 | [stage-layout.md](stage-layout.md) |
| 写程序·语法 | [arkui-syntax.md](arkui-syntax.md) |
| 写程序·视觉 | [ui-style.md](ui-style.md) |

## 落盘 ↔ 官网

| 本目录文件 | 用途 | 官网 |
|------------|------|------|
| [js-apis-pipWindow.md](js-apis-pipWindow.md) | 画中画接口 | https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-pipwindow |
| [js-apis-floatView.md](js-apis-floatView.md) | 闪控窗接口（含 bind/unbind） | https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-floatview |
| [js-apis-floatingBall.md](js-apis-floatingBall.md) | 闪控球接口 | https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-floatingball |
| [window-type-overview.md](window-type-overview.md) | 窗口类型 / 球窗对比 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/window-type-overview |
| [guide-float-view.md](guide-float-view.md) | 闪控窗开发指导（含球窗绑定、与防窥组合） | https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/float-view-guide |
| [guide-dlp-anti-peep.md](guide-dlp-anti-peep.md) | 防窥保护开发指导 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/devicesecurity-dlpantipeep |
| [errorcode-window-float.md](errorcode-window-float.md) | 窗口错误码 13000xx | 官网窗口错误码页对应章节 |
| [restricted-permissions-float.md](restricted-permissions-float.md) | FLOAT_VIEW / USE_FLOAT_BALL / DLP_GET_HIDE_STATUS 等 | 官网受限权限页对应条目 |
| [declare-permissions.md](declare-permissions.md) | module.json5 声明 | 官网「声明权限」 |
| [declare-permissions-in-acl.md](declare-permissions-in-acl.md) | ACL | 官网「申请受限权限」 |
| [stage-layout.md](stage-layout.md) | Stage 单模块工程树与落点 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/start-with-ets-stage 、https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-package-structure-stage |
| [arkui-syntax.md](arkui-syntax.md) | 基本语法 + 装饰器总表 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-basic-syntax-overview 、https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-decorator-overview |
| [ui-style.md](ui-style.md) | 系统默认视觉（分层参数） | OpenHarmony UX 视觉基础/色彩/按钮；实现用 `sys.color` / `sys.float` |

## 本仓实现对照（选读，非权威）

仅查「框架是否拒绝 / 日志从哪打」时读。不要用这些文件覆盖官网签名。

| 用途 | 仓库路径 |
|------|----------|
| 闪控窗控制器 | `window_window_manager/wm/src/float_view_controller.cpp` |
| 绑定 | `window_window_manager/wm/src/float_window_manager.cpp` |
| 闪控球 | `window_window_manager/wm/src/floating_ball_controller.cpp` |
| 查错 YAML | `Watchman/modules/pip/module.yaml`、`float_view/module.yaml`、`floating_ball/module.yaml` |
