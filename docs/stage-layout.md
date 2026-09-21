# Stage 工程落点（写程序必读）

权威：[构建第一个 ArkTS 应用（Stage）](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/start-with-ets-stage)、[应用程序包结构](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-package-structure-stage)。本页是摘要，不是官网全文。不要用 JS Lite / FA / `config.json`。

仅 Stage 单模块。不生成 HAP/HAR/HSP、hvigor、Native 工程。

## 目录（Empty Ability）

```text
Project/
├── AppScope/
│   └── app.json5                          # bundleName / 版本；写悬浮窗一般不改
├── entry/                                 # 模块名以工程为准，下文用 entry
│   ├── src/main/
│   │   ├── ets/
│   │   │   ├── entryability/EntryAbility.ets
│   │   │   └── pages/
│   │   │       ├── Index.ets              # 默认把 PiP / 球 / 启动闪控窗写这里
│   │   │       └── FloatPanel.ets         # 仅闪控窗：setUIContext 加载的页
│   │   ├── resources/base/
│   │   │   ├── profile/main_pages.json    # 页面路径表
│   │   │   └── element/string.json        # FLOAT_VIEW 的 reason
│   │   └── module.json5                   # requestPermissions
│   └── build-profile.json5                # 不要编造；用户没要脚手架就别生成
└── build-profile.json5
```

`AppScope` 目录名不要改。模块目录可以不叫 `entry`，落点相对 `src/main/` 不变。

## 改哪些文件

| 路径 | 职责 | 写悬浮窗时 |
|------|------|------------|
| `ets/pages/Index.ets` | `@Entry` 页面 | PiP / 闪控球 / 点按钮 start 闪控窗 |
| `ets/pages/FloatPanel.ets` | 闪控窗内容页 | `setUIContext('pages/FloatPanel')` 的目标；也要 `@Entry` |
| `resources/base/profile/main_pages.json` | 页面注册 | 路径须与 `setUIContext` / `loadContent` 的字符串一致，禁止相对路径 |
| `module.json5` | 模块配置 + 权限 | `requestPermissions`；详见 [declare-permissions.md](declare-permissions.md) |
| `resources/*/element/string.json` | 字符串 | `FLOAT_VIEW` 的 `reason` |
| 签名 Profile | ACL | 调试签名；详见 [declare-permissions-in-acl.md](declare-permissions-in-acl.md) |
| `ets/entryability/EntryAbility.ets` | Ability | `onWindowStageCreate` → `windowStage.loadContent('pages/Index')`。主窗 show 在这之后，不在页面 `aboutToAppear` |

`main_pages.json` 示例：

```json
{
  "src": [
    "pages/Index",
    "pages/FloatPanel"
  ]
}
```

`module.json5` 里对应 `"pages": "$profile:main_pages"`。

## 产出规则

默认 **直接改用户工程里的文件**（读现有内容再补丁进去）。禁止只在对话里讲「请把下面代码拷到 Xxx.ets」。对话里可以简述改了哪几处，代码必须以文件修改落地。

禁止往 `window_window_manager/`、`foundation/`、`interface/` 等框架源码里写应用 ArkTS。本 Skill 只改 **应用工程**。

1. **已有应用工程**：工作区存在 `**/src/main/ets/pages/*.ets`（或用户给出模块路径）。改现有 `Index.ets`、`module.json5`、`main_pages.json` 等。闪控窗内容页不存在时才 **新建** `FloatPanel.ets` 并登记 `main_pages.json`。不要另建 `PipDemo.ets` / `BallPage.ets` 一套平行页面，除非用户点名要新页。
2. **找不到应用页**：`**/src/main/ets/pages` 不存在（常见：工作区是 OHOS 源码仓、只有 WMS）。**停手问用户应用工程路径**。不要在当前仓根下新建 `entry/`，不要把 demo 写进框架目录。
3. **用户明确只要脚手架、且已给应用路径**：按上表在该路径 **创建** `entry/src/main/...`。只建页面 `.ets` + `main_pages.json` + 权限相关 JSON。不要生成 `oh-package.json5` / 工程级 `build-profile.json5` / 全量脚手架，除非用户明确要工程。
4. 每处修改对应真实路径。不要只丢无路径代码块让用户自己贴。

## 生命周期（和铁律对齐）

| 时机 | 在哪 | 能否 start PiP/窗/球 |
|------|------|----------------------|
| `UIAbility.onCreate` | Ability | 否 |
| `aboutToAppear` | 页面组件 | 否（主窗未必已 show） |
| `onWindowStageCreate` → `loadContent` 完成、主窗前台 | Ability | 可以；推荐按钮 / 主窗已显示后的回调 |
