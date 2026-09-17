# 闪控窗开发指导（Skill 落盘）

官网：https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/float-view-guide  
锚点「复杂场景与防窥保护组合使用」：https://developer.huawei.com/consumer/cn/doc/HarmonyOS-Guides/float-view-guide#%E5%A4%8D%E6%9D%82%E5%9C%BA%E6%99%AF%E4%B8%8E%E9%98%B2%E7%AA%A5%E4%BF%9D%E6%8A%A4%E7%BB%84%E5%90%88%E4%BD%BF%E7%94%A8

完整接口签名、示例代码以同目录 [js-apis-floatView.md](js-apis-floatView.md)、[js-apis-floatingBall.md](js-apis-floatingBall.md) 为准。刷新时用官网指导页覆盖本文。

## 选型

见 [window-type-overview.md](window-type-overview.md)。

- 视频画面迁到系统小窗、系统画控制条 → 画中画，不是闪控窗
- 贴边小球、不能自定义 UI → 闪控球
- 系统画窗框、应用 `setUIContext` 加载自己的页 → 闪控窗
- 点球展开窗、点窗缩小回球 → `floatView.bind`
- `TYPE_FLOAT` 全局悬浮窗、`createSubWindow` 子窗：不是本指导范围

## 开发步骤

1. `canIUse('SystemCapability.Window.SessionManager')` 且 `floatView.isFloatViewEnabled()`
2. 权限见 [restricted-permissions-float.md](restricted-permissions-float.md) + [declare-permissions.md](declare-permissions.md) + [declare-permissions-in-acl.md](declare-permissions-in-acl.md)：`ohos.permission.FLOAT_VIEW`（user_grant，要 reason/usedScene 和运行时弹窗）；绑定还要 `ohos.permission.USE_FLOAT_BALL`（system_grant，不弹窗）
3. `floatView.create(config)` 只拿控制器，不建窗
4. `setUIContext(path)` 或 `setUIContextByName`；path 与 `main_pages.json` 的 src 一致
5. `onStateChange`；主窗前台后 `start()`
6. **`start()` 的 Promise 返回不表示启动完成**，以回调 `STARTED` 为准（见接口文档）
7. 退出 `stop` + `offStateChange`

## 球窗绑定

见接口文档 `floatView.bind` / `unbind`。

- 两边都 `create` 且都未 start、未绑定再 bind
- 之后 `start()` 或 `startFloatingBall()` 会同时创建两个窗口，同一时刻只展示一个；先调谁谁先亮
- 用户点窗左上角缩小 ↔ 点球展开，由系统切换
- 窗状态 `IN_FLOATING_BALL`；停因 `FLOATING_BALL_STOP` / `FLOAT_VIEW_STOP`
- `stop` 任一即两边一起停；都停下后才能 unbind
- 权限：`FLOAT_VIEW` + `USE_FLOAT_BALL`

未绑定同时 start 窗和球、或与已启动 PiP 并行 → `1300034`，见 [errorcode-window-float.md](errorcode-window-float.md)。

## 复杂场景与防窥保护组合使用

闪控窗负责持续展示；防窥保护负责窥视时保护敏感内容。两套能力组合，不要在 `FloatViewController` 上编防窥方法。

防窥接口与示例见 [guide-dlp-anti-peep.md](guide-dlp-anti-peep.md)。组合要点：

1. 声明 `ohos.permission.DLP_GET_HIDE_STATUS`（受限、system_grant、ACL），与闪控窗权限并列
2. `canIUse('SystemCapability.Security.DlpAntiPeep')`
3. `isDlpAntiPeepSwitchOn()`；未开则 `requestAntiPeepOptions(context)`（设置 → 隐私与安全 → 防窥保护）
4. 闪控窗 `STARTED` 之后再 `on('dlpAntiPeep')`
5. 状态 `HIDE` 时：
   - 闪控窗页面把敏感字段改为占位（如 `****`），不要只依赖蒙层
   - `setAntiPeepMaskLayer(windowId)` 的 **windowId 必须是闪控窗** `getWindowProperties().windowId`，不要用 `window.getLastWindow` / 主窗 id（小窗叠在其它应用上时蒙主窗挡不住）
6. 绑定了闪控球时，同步 `updateFloatingBall` 脱敏 title/content
7. `PASS` 时恢复展示；页面销毁 / 闪控窗 `STOPPED` 时 `off('dlpAntiPeep')`
8. 同一窗口不要反复拉蒙层，用标志位

错误码见防窥指导；闪控窗 `getWindowProperties` 须已 start，否则 `1300031`。
