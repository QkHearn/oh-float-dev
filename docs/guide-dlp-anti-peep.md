# 防窥保护开发指导（Skill 落盘）

官网：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/devicesecurity-dlpantipeep  
与闪控窗组合：见 [guide-float-view.md](guide-float-view.md)「复杂场景与防窥保护组合使用」。

刷新时用官网现行页覆盖本文。

## 能力与权限

- 导入：`import { dlpAntiPeep } from '@kit.DeviceSecurityKit'`
- syscap：`SystemCapability.Security.DlpAntiPeep`，调用前 `canIUse`，不支持则整段跳过
- 权限：`ohos.permission.DLP_GET_HIDE_STATUS`（见 [restricted-permissions-float.md](restricted-permissions-float.md)）
  - 级别 system_basic，授权方式 system_grant（声明 + ACL，不弹窗）
  - 支持设备：Phone；API20 起对普通应用开放
- 用户须在「设置 > 隐私与安全 > 防窥保护」打开本应用

## 接口

| 接口 | 作用 |
|------|------|
| `isDlpAntiPeepSwitchOn(): Promise<boolean>` | 本应用是否已开防窥保护 |
| `requestAntiPeepOptions(context): Promise<...>` | 拉起设置弹窗请用户打开开关 |
| `on('dlpAntiPeep', callback)` | 订阅窥视状态 |
| `off('dlpAntiPeep', callback?)` | 取消订阅 |
| `getDlpAntiPeepInfo(): DlpAntiPeepStatus` | 同步当前窥视状态 |
| `setAntiPeepMaskLayer(windowId: number): Promise<void>` | 对指定窗口拉起系统级蒙层 |
| `passDlpAntiPeepInfo(): void` | 直到锁屏或退出前视为非窥视 |
| `publishAntiPeepInformation(): Promise<void>` | 发布防窥实况窗提醒 |

## 状态

| `DlpAntiPeepStatus` | 含义 | 应用 |
|---------------------|------|------|
| `PASS` | 无人窥视（或仅机主） | 显示真数据，可清蒙层标志 |
| `HIDE` | 机主与非机主同时看屏 | 藏敏感字段 + 按需蒙层 |

## 推荐顺序

```text
canIUse → isDlpAntiPeepSwitchOn
  → 未开：requestAntiPeepOptions
  → getDlpAntiPeepInfo 同步一次
  → on('dlpAntiPeep')
  → HIDE：脱敏 + setAntiPeepMaskLayer(目标窗口 windowId)
  → PASS：恢复
  → aboutToDisappear：off
```

`setAntiPeepMaskLayer` 的 windowId：

- 主界面：主窗 `getWindowProperties().id`
- **闪控窗场景：闪控窗 `FloatViewProperties.windowId`**（`STARTED` 之后 `getWindowProperties()`）

不要对同一 windowId 在一次 HIDE 期间连打蒙层。

## 与闪控窗

防窥不是 `floatView` 的 API。组合规则写在 [guide-float-view.md](guide-float-view.md)。本仓 OpenHarmony 树可能没有 `@kit.DeviceSecurityKit` DTS，**仍按本指导写应用侧示例**（以官网为准）。
