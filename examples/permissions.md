# 权限（窗 / 球 / 防窥）

纯 PiP 可跳过本文件。资料：[docs/declare-permissions.md](../docs/declare-permissions.md)、[docs/declare-permissions-in-acl.md](../docs/declare-permissions-in-acl.md)、[docs/restricted-permissions-float.md](../docs/restricted-permissions-float.md)。

**1. 声明** — 目标模块 `src/main/module.json5` 的 `requestPermissions`（按能力裁剪）：

```json5
"requestPermissions": [
  {
    "name": "ohos.permission.FLOAT_VIEW",
    "reason": "$string:float_view_reason",
    "usedScene": { "abilities": ["EntryAbility"], "when": "inuse" }
  },
  {
    "name": "ohos.permission.USE_FLOAT_BALL"
  },
  {
    "name": "ohos.permission.DLP_GET_HIDE_STATUS"
  }
]
```

`FLOAT_VIEW` 的 `reason` 配在同模块 `resources/*/element/string.json`。

**2. ACL** — 调试签名 Profile。`apl` 可仍为 `normal`（DevEco 自动签常见如此），关键是 `allowed-acls` 三条。不要改已签名 p7b 的 `apl` 去凑 `system_basic`。

```json5
"bundle-info": { "apl": "normal" },
"acls": {
  "allowed-acls": [
    "ohos.permission.FLOAT_VIEW",
    "ohos.permission.USE_FLOAT_BALL",
    "ohos.permission.DLP_GET_HIDE_STATUS"
  ]
}
```

**3. 运行时** — 只对 **user_grant 的 `FLOAT_VIEW`** 弹窗。`USE_FLOAT_BALL`、`DLP_GET_HIDE_STATUS` 是 system_grant，声明+ACL 后安装即授，不要弹窗。

**4. 防窥开关（不是权限弹窗）** — ACL ≠ 系统「防窥保护」。原因见 [guide-dlp-anti-peep.md](../docs/guide-dlp-anti-peep.md)，代码见 [peep.md](peep.md)。

```ts
import { abilityAccessCtrl, common, Permissions } from '@kit.AbilityKit';
import { BusinessError } from '@kit.BasicServicesKit';

async function requestPerms(ctx: common.UIAbilityContext, perms: Permissions[]): Promise<boolean> {
  try {
    const atManager = abilityAccessCtrl.createAtManager();
    const grant = await atManager.requestPermissionsFromUser(ctx, perms);
    return grant.authResults.every((v: number) => v === 0);
  } catch (e) {
    const err = e as BusinessError;
    console.error(`requestPerms ${err.code} ${err.message}`);
    return false;
  }
}
```
