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

1. **已有工程**：按上表改现有文件，路径跟用户工程走。不要新建一套工程。
2. **没有工程**：仍按上表路径给补丁（标 `entry/src/main/...`）。只给页面 `.ets` + `main_pages.json` 增量 + 权限 JSON。不要生成 `oh-package.json5` / 工程级 `build-profile.json5` / 全量脚手架，除非用户明确要工程。
3. 产出必须带路径，例如 `entry/src/main/ets/pages/Index.ets`，不要只丢无路径代码块。

## 生命周期（和铁律对齐）

| 时机 | 在哪 | 能否 start PiP/窗/球 |
|------|------|----------------------|
| `UIAbility.onCreate` | Ability | 否 |
| `aboutToAppear` | 页面组件 | 否（主窗未必已 show） |
| `onWindowStageCreate` → `loadContent` 完成、主窗前台 | Ability | 可以；推荐按钮 / 主窗已显示后的回调 |
