# 可抄骨架

生成代码时按需求改文案/页面路径/模板，**不要改调用顺序**。API 名以 [docs/](docs/README.md) 落盘接口文档为准。

权限要写两到三处（资料：[docs/declare-permissions.md](docs/declare-permissions.md)、[docs/declare-permissions-in-acl.md](docs/declare-permissions-in-acl.md)、[docs/restricted-permissions-float.md](docs/restricted-permissions-float.md)）。

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

**2. ACL** — 调试签名 Profile（SDK `toolchains/lib/UnsgnedReleasedProfileTemplate.json` 或 DevEco 签名配置）加入：

```json5
"bundle-info": { "apl": "system_basic" },
"acls": {
  "allowed-acls": [
    "ohos.permission.FLOAT_VIEW",
    "ohos.permission.USE_FLOAT_BALL",
    "ohos.permission.DLP_GET_HIDE_STATUS"
  ]
}
```

**3. 运行时** — 只对 **user_grant 的 `FLOAT_VIEW`** 弹窗；`USE_FLOAT_BALL`、`DLP_GET_HIDE_STATUS` 是 system_grant，声明+ACL 后安装即授，不要对它们弹窗。防窥组合按需声明 `DLP_GET_HIDE_STATUS`，纯 PiP 可不写上述权限。

```ts
import { abilityAccessCtrl, common, Permissions } from '@kit.AbilityKit';
import { BusinessError } from '@kit.BasicServicesKit';

async function requestPerms(ctx: common.UIAbilityContext, perms: Permissions[]): Promise<boolean> {
  const atManager = abilityAccessCtrl.createAtManager();
  const grant = await atManager.requestPermissionsFromUser(ctx, perms);
  return grant.authResults.every((v: number) => v === 0);
}
```

---

## 画中画（API12+ typeNode，推荐）

页面不必把 XComponent 放进当前布局。`create(config, contentNode)`；`updateContentNode` 仅此路径。指导全文未落盘，签名以 `docs/js-apis-pipWindow.md` 的 `create12` 节为准。

```ts
import { PiPWindow, typeNode } from '@kit.ArkUI';
import { BusinessError } from '@kit.BasicServicesKit';
import { common } from '@kit.AbilityKit';

@Entry
@Component
struct PipTypeNodePage {
  private xCtrl: XComponentController = new XComponentController();
  private pip?: PiPWindow.PiPController;
  private node?: typeNode.XComponent;

  private async start(): Promise<void> {
    if (!canIUse('SystemCapability.Window.SessionManager') || !PiPWindow.isPiPEnabled()) {
      return;
    }
    const uiCtx = this.getUIContext();
    const ctx = uiCtx.getHostContext() as common.UIAbilityContext;
    const options: XComponentOptions = { type: XComponentType.SURFACE, controller: this.xCtrl };
    this.node = typeNode.createNode(uiCtx, 'XComponent', options);
    const config: PiPWindow.PiPConfiguration = {
      context: ctx,
      componentController: this.xCtrl,
      templateType: PiPWindow.PiPTemplateType.VIDEO_PLAY,
      contentWidth: 1920,
      contentHeight: 1080
    };
    try {
      this.pip = await PiPWindow.create(config, this.node);
      this.pip.on('stateChange', (state: PiPWindow.PiPState, reason: string) => {
        console.info(`pip state=${state} reason=${reason}`);
      });
      this.pip.on('controlEvent', (event: PiPWindow.ControlEventParam) => {
        console.info(`pip control=${event.controlType}`);
      });
      await this.pip.startPiP(); // 主窗已 show 后再调
    } catch (e) {
      const err = e as BusinessError;
      console.error(`pip ${err.code} ${err.message}`);
    }
  }

  aboutToDisappear(): void {
    this.pip?.off('stateChange');
    this.pip?.off('controlEvent');
  }

  build() {
    Button('start PiP').onClick(() => { this.start(); })
  }
}
```

---

## 画中画（页面已有 XComponent）

```ts
import { PiPWindow } from '@kit.ArkUI';
import { BusinessError } from '@kit.BasicServicesKit';
import { common } from '@kit.AbilityKit';

@Entry
@Component
struct PipPage {
  private xCtrl: XComponentController = new XComponentController();
  private pip?: PiPWindow.PiPController;
  private starting: boolean = false;

  aboutToAppear(): void {
    // 不要在这里 startPiP
  }

  private async ensureCtrl(): Promise<boolean> {
    if (!canIUse('SystemCapability.Window.SessionManager') || !PiPWindow.isPiPEnabled()) {
      console.error('PiP not supported');
      return false;
    }
    if (this.pip) {
      return true;
    }
    const ctx = this.getUIContext().getHostContext() as common.UIAbilityContext;
    const config: PiPWindow.PiPConfiguration = {
      context: ctx,
      componentController: this.xCtrl, // 必须与下方 XComponent 同一个
      navigationId: 'nav_pip',
      templateType: PiPWindow.PiPTemplateType.VIDEO_PLAY,
      contentWidth: 1920,
      contentHeight: 1080,
      controlGroups: [PiPWindow.VideoPlayControlGroup.VIDEO_PREVIOUS_NEXT]
    };
    try {
      this.pip = await PiPWindow.create(config);
      this.pip.on('stateChange', (state: PiPWindow.PiPState, reason: string) => {
        console.info(`pip state=${state} reason=${reason}`);
        this.starting = false;
      });
      this.pip.on('controlEvent', (event: PiPWindow.ControlEventParam) => {
        console.info(`pip control=${event.controlType} status=${event.status}`);
      });
      return true;
    } catch (e) {
      const err = e as BusinessError;
      console.error(`create pip ${err.code} ${err.message}`);
      return false;
    }
  }

  private async start(): Promise<void> {
    if (this.starting) {
      return;
    }
    this.starting = true;
    if (!await this.ensureCtrl() || !this.pip) {
      this.starting = false;
      return;
    }
    try {
      await this.pip.startPiP();
    } catch (e) {
      this.starting = false;
      const err = e as BusinessError;
      // 1300013 主窗未 show；1300015 重复；1300034 与闪控窗冲突
      console.error(`start pip ${err.code} ${err.message}`);
    }
  }

  aboutToDisappear(): void {
    this.pip?.off('stateChange');
    this.pip?.off('controlEvent');
  }

  build() {
    Navigation() {
      Column() {
        XComponent({ id: 'video', type: XComponentType.SURFACE, controller: this.xCtrl })
          .width('100%').height(200)
        Button('start PiP').onClick(() => { this.start(); })
        Button('stop PiP').onClick(() => { this.pip?.stopPiP(); })
      }
    }.id('nav_pip')
  }
}
```

---

## 闪控窗

```ts
import { floatView } from '@kit.ArkUI';
import { BusinessError } from '@kit.BasicServicesKit';
import { common } from '@kit.AbilityKit';

@Entry
@Component
struct FloatViewPage {
  private fv?: floatView.FloatViewController;
  private started: boolean = false;

  private async start(): Promise<void> {
    if (!canIUse('SystemCapability.Window.SessionManager') || !floatView.isFloatViewEnabled()) {
      return;
    }
    const ctx = this.getUIContext().getHostContext() as common.UIAbilityContext;
    if (!await requestPerms(ctx, ['ohos.permission.FLOAT_VIEW'])) {
      return;
    }
    try {
      this.fv = await floatView.create({
        context: ctx,
        templateType: floatView.FloatViewTemplateType.ROUNDED_RECTANGLE
      });
      this.fv.onStateChange((info) => {
        // 以回调为准，不要用 start() Promise 当显示成功
        this.started = true;
        console.info(`fv state=${JSON.stringify(info)}`);
      });
      await this.fv.setUIContext('pages/FloatPanel'); // 必须写入 main_pages.json
      await this.fv.start();
    } catch (e) {
      const err = e as BusinessError;
      // 201 权限；1300033 主窗非前台；1300034 与 PiP/球冲突
      console.error(`fv ${err.code} ${err.message}`);
    }
  }

  aboutToDisappear(): void {
    this.fv?.offStateChange();
    if (this.started) {
      this.fv?.stop();
    }
  }

  build() {
    Button('start float view').onClick(() => { this.start(); })
  }
}
```

`pages/FloatPanel` 里的可点控件避开 `getWindowProperties().avoidArea`。

---

## 闪控球

```ts
import { floatingBall } from '@kit.ArkUI';
import { BusinessError } from '@kit.BasicServicesKit';
import { common } from '@kit.AbilityKit';

@Entry
@Component
struct BallPage {
  private fb?: floatingBall.FloatingBallController;
  private readonly tpl: floatingBall.FloatingBallTemplate = floatingBall.FloatingBallTemplate.EMPHATIC;

  private async start(): Promise<void> {
    if (!floatingBall.isFloatingBallEnabled()) {
      return;
    }
    const ctx = this.getUIContext().getHostContext() as common.UIAbilityContext;
    // USE_FLOAT_BALL 为 system_grant：module.json5 + ACL 即可，不要弹窗
    try {
      this.fb = await floatingBall.create({ context: ctx });
      this.fb.on('stateChange', (state: floatingBall.FloatingBallState) => {
        console.info(`fb state=${state}`);
      });
      this.fb.on('click', () => {
        this.fb?.restoreMainWindow();
      });
      await this.fb.startFloatingBall({
        template: this.tpl,
        title: '盯盘',
        content: '沪指 +0.8%'
      });
    } catch (e) {
      const err = e as BusinessError;
      console.error(`fb ${err.code} ${err.message}`);
    }
  }

  private async refresh(text: string): Promise<void> {
    // template 必须与 start 相同；STATIC 不要走这里
    await this.fb?.updateFloatingBall({ template: this.tpl, title: '盯盘', content: text });
  }
}
```

---

## 窗 + 球绑定

两边都 `create` 且**都未 start**，再 bind，再 start 其中一个。

```ts
import { floatView, floatingBall } from '@kit.ArkUI';
import { common } from '@kit.AbilityKit';

async function bindAndStart(host: Object): Promise<void> {
  const page = host as FloatViewPage; // 示意：持有两个 controller 的组件
  const ctx = page.getUIContext().getHostContext() as common.UIAbilityContext;
  if (!await requestPerms(ctx, ['ohos.permission.FLOAT_VIEW'])) {
    return;
  }
  const fv = await floatView.create({
    context: ctx,
    templateType: floatView.FloatViewTemplateType.ROUNDED_RECTANGLE
  });
  const fb = await floatingBall.create({ context: ctx });
  const ballParams: floatingBall.FloatingBallParams = {
    template: floatingBall.FloatingBallTemplate.SIMPLE,
    title: '回到面板'
  };
  await floatView.bind(fv, fb, ballParams);
  await fv.setUIContext('pages/FloatPanel');
  await fv.start(); // 同时建窗和球，先展示窗；用户点缩小变球
}
```

---

## 闪控窗 + 防窥保护

指导：[docs/guide-float-view.md](docs/guide-float-view.md) 组合节、[docs/guide-dlp-anti-peep.md](docs/guide-dlp-anti-peep.md)。须已 `STARTED`，windowId 用闪控窗的，不要用主窗。

```ts
import { dlpAntiPeep } from '@kit.DeviceSecurityKit';
import { floatView } from '@kit.ArkUI';
import { common } from '@kit.AbilityKit';
import { BusinessError } from '@kit.BasicServicesKit';

let peepOn: boolean = false;
let maskApplied: boolean = false;

async function attachPeep(fv: floatView.FloatViewController, ctx: common.UIAbilityContext): Promise<void> {
  if (!canIUse('SystemCapability.Security.DlpAntiPeep')) {
    return;
  }
  const on = await dlpAntiPeep.isDlpAntiPeepSwitchOn();
  if (!on) {
    await dlpAntiPeep.requestAntiPeepOptions(ctx);
    return;
  }
  const props = fv.getWindowProperties();
  const windowId = props.windowId;
  dlpAntiPeep.on('dlpAntiPeep', (status) => {
    if (status === dlpAntiPeep.DlpAntiPeepStatus.HIDE) {
      // 页面敏感字段改为 ****；绑定球时同步 updateFloatingBall 脱敏
      if (!maskApplied) {
        dlpAntiPeep.setAntiPeepMaskLayer(windowId).then(() => {
          maskApplied = true;
        }).catch((e: BusinessError) => {
          console.error(`mask ${e.code} ${e.message}`);
        });
      }
    } else {
      maskApplied = false;
    }
  });
  peepOn = true;
}

function detachPeep(): void {
  if (peepOn) {
    dlpAntiPeep.off('dlpAntiPeep');
    peepOn = false;
    maskApplied = false;
  }
}
```

在闪控窗 `onStateChange` 收到 `STARTED` 后调 `attachPeep`；收到 `STOPPED` 或页面销毁时调 `detachPeep`。

错误示范（生成代码时禁止）：

- create 之后立刻当窗已显示
- 先 `startFloatingBall` 再 `bind`
- PiP `startPiP` 与 `fv.start` 并行
- 闪控球 `updateFloatingBall` 换 template 或对 STATIC 更新
- 编造 `fv.setAntiPeep(...)`
- `setAntiPeepMaskLayer` 传入 `window.getLastWindow` / 主窗 id
