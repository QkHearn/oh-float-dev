# 三种能力：页面怎么加载

Ability 只 `loadContent('pages/Index')`。每种能力自己的页必须进 `main_pages.json`，字符串和加载 API **逐字相同**，否则主页空白、二级页白屏、闪控窗空框。

路径写 `pages/Xxx`，禁止 `./pages/Xxx`。`module.json5` 用 `"pages": "$profile:main_pages"`。

| 能力 | 谁画界面 | 必须登记 | 怎么加载到屏上 |
|------|----------|----------|----------------|
| 闪控球 | 系统画贴边球，应用**没有球页面** | 只要主页 `pages/Index` | Index 按钮 `startFloatingBall`。禁止 `setUIContext`、禁止新建球页 |
| 画中画 | 应用视频页 + 系统小窗 | 宿主 `@Entry`（`Index` 或 `PipHost`） | 见 [pip.md](pip.md)。`Page1` 根节点必须是 `NavDestination()` |
| 闪控窗 | 另一份 `@Entry` 内容页 | `pages/Index` **和** `pages/FloatPanel` | Index `setUIContext('pages/FloatPanel')` 再 `start()`。窗里加载的**不是** Index |

```json
{
  "src": [
    "pages/Index",
    "pages/PipHost",
    "pages/FloatPanel"
  ]
}
```

只做一条就只登记那条用到的 `@Entry`。没登记的 `@Entry`：`router.pushUrl` / `setUIContext` / `loadContent` 都会失败。`Page1` 不是 `@Entry`，不要登记。

不要三条同时 `start`（`1300034`）。球↔窗 [bind.md](bind.md)；窗+防窥 [peep.md](peep.md)。画中画+球窗见下方「叠加 Index」。

---

## 叠加 Index（画中画 + 球窗，可选防窥）

同一 demo 可以同时有会议画中画和球窗，**运行时只 start 一套**。登记 `pages/Index`、`pages/PipHost`、`pages/FloatPanel`。

- **进入会议**：`this.getUIContext().getRouter().pushUrl({ url: 'pages/PipHost' })`（PiP 尚未 start）。球窗已 start 则不准进，上屏「请先停止球窗」
- **启动球窗**：`floatView.bind` 后 `start()`。`1300034` 上屏「请先停止画中画」
- 强调按钮只留一个主入口（进入会议）；启动/停止球窗、拉起蒙层用灰底次按钮
- 不要同时 `startPiP` 和 `floatView.start`

防窥：[peep.md](peep.md)。Index `aboutToAppear` 就监听，不要等球窗 STARTED。

---

## 闪控球

读 [floating-ball.md](floating-ball.md)。权限 [permissions.md](permissions.md)（只声明+ACL，不弹窗）。

球没有应用页面。Index 只负责 `create` → 按钮 `startFloatingBall` → 等 `STARTED`。系统按模板画球，不能自绘。

加载失败常见原因：当成闪控窗去 `setUIContext`；新建 `BallPage` 当球 UI；在 `aboutToAppear` 里 start。

---

## 画中画

读 [pip.md](pip.md) 选 **一条**，默认再读 [pip-xcomponent.md](pip-xcomponent.md)。

写新 demo：宿主 `Navigation` `pushPath` → `Page1`。`navigationId` === `Navigation.id`。二级页白屏 = `PageMap` 根节点不是 `NavDestination()`。还原失败 / `1300013` = 两个 id 不是同一字符串，或单页却写了 `navigationId`。

已有单页、没有 Navigation：画面做在这一页，不要包 `Navigation`，不要写 `navigationId`。细则：[docs/guide-pip.md](../docs/guide-pip.md)「Navigation 避坑」。

---

## 闪控窗

读 [float-view.md](float-view.md)。权限 [permissions.md](permissions.md)（`FLOAT_VIEW` 要弹窗）。

两份 `@Entry`，两套实例：

1. **Index**：`create` 一次 → `setUIContext('pages/FloatPanel')` → 按钮 `start()` → 等回调 `STARTED`（Promise 返回不算起来）
2. **FloatPanel**：只有窗内 UI，必须 `@Entry`，必须已在 `main_pages.json`

`setUIContext` 的字符串 = `main_pages.json` 的 `src`。写 `pages/Index`、相对路径、或没登记 `FloatPanel`：窗框在、内容空。不要把启动逻辑写进 FloatPanel。退后台要继续展示：不要在 `aboutToDisappear` / `onPageHide` 里 `stop`。
