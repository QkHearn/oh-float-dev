# 可抄骨架

路径：[docs/stage-layout.md](docs/stage-layout.md)。语法：[docs/arkui-syntax.md](docs/arkui-syntax.md)。视觉：[docs/ui-style.md](docs/ui-style.md)。

下面是往 **用户工程现有文件里合并** 的骨架，不是另存一份新 demo 文件。有 `Index.ets` 就改它的 `start`/`build`；闪控窗没有 `FloatPanel.ets` 才新建。按需求改文案/页面路径/**模板**，**不要改调用顺序**。用户说了某一种样式，按 `SKILL.md`「模板对照」改枚举。

API 名以 [docs/](docs/README.md) 落盘接口文档为准。签名对不上再 Grep 对应 `js-apis-*.md` 该节。

**必须包含**（抄骨架前核对）：

1. 能力探测：syscap + `isXxxEnabled`
2. 权限三处：下方 `module.json5` + ACL；仅 `FLOAT_VIEW` 弹窗。PiP 无特殊权限。`USE_FLOAT_BALL` / `DLP_GET_HIDE_STATUS` 不弹窗
3. 调用顺序与 `BusinessError` 与本节骨架一致
4. 状态回调：PiP `on('stateChange')`；窗 `onStateChange`；球 `on('stateChange'|'click')`
5. 退出：`stop*` + `off*`
6. PiP API12+ 默认 typeNode 骨架；页面已有 XComponent 用「页面已有 XComponent」节
7. 球↔窗：`floatView.bind`，听窗状态 `IN_FLOATING_BALL`；不要自己 start 两套。细节：[reference-float-view.md](reference-float-view.md) 绑定节
8. 闪控窗防窥：[docs/guide-float-view.md](docs/guide-float-view.md) 组合节 + [docs/guide-dlp-anti-peep.md](docs/guide-dlp-anti-peep.md)。不要编 `FloatViewController` 防窥 API；蒙层 windowId = 闪控窗 `getWindowProperties().windowId`

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

## 合并进现有 Index（先看这段）

下面各节的 `@Entry struct XxxPage` **只作对照**，不要另存为 `PipTypeNodePage.ets` / `FloatViewPage.ets` / `BallPage.ets`。落到用户工程时：

1. 把 import、字段、`start()`、`aboutToDisappear()`、`build()` 里的按钮 **并入现有 `Index`**
2. 已有 `build()` 就加一个按钮，不要换掉整页 UI
3. 只有闪控窗内容页不存在时才新建 `FloatPanel.ets`

最小合并（画中画；窗/球同理，只并方法，不换 struct 名）：

```ts
// 已有 Index.ets：保留原有字段和布局，追加 import / 字段 / startPip / 按钮
import { PiPWindow, typeNode } from '@kit.ArkUI';
import { BusinessError } from '@kit.BasicServicesKit';
import { common } from '@kit.AbilityKit';

@Entry
@Component
struct Index {
  private xCtrl: XComponentController = new XComponentController();
  private pip?: PiPWindow.PiPController;
  private node?: typeNode.XComponent;

  private async startPip(): Promise<void> {
    // 体见下一节 start()，只改模板枚举
  }

  aboutToDisappear(): void {
    this.pip?.off('stateChange');
    this.pip?.off('controlEvent');
    this.pip?.stopPiP();
  }

  build() {
    Column() {
      Button('进入画中画')
        .onClick(() => { this.startPip(); })
    }
    .width('100%')
  }
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
    Column() {
      Text('画中画')
        .fontSize(20)
        .fontWeight(FontWeight.Medium)
        .fontColor($r('sys.color.ohos_id_color_text_primary'))
        .width('100%')
      Text('退出后以系统小窗继续播放')
        .fontSize(14)
        .fontColor($r('sys.color.ohos_id_color_text_secondary'))
        .margin({ top: 8 })
        .width('100%')
      Blank()
      Button('启动')
        .type(ButtonType.Capsule)
        .height(40)
        .width('100%')
        .fontSize(16)
        .fontColor(Color.White)
        .backgroundColor($r('sys.color.ohos_id_color_emphasize'))
        .onClick(() => { this.start(); })
    }
    .width('100%')
    .height('100%')
    .padding({ left: 16, right: 16, top: 24, bottom: 24 })
    .backgroundColor($r('sys.color.ohos_id_color_background'))
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
          .width('100%')
          .height(200)
          .borderRadius($r('sys.float.ohos_id_corner_radius_card'))
          .clip(true)
        Button('启动')
          .type(ButtonType.Capsule)
          .height(40)
          .width('100%')
          .fontSize(16)
          .fontColor(Color.White)
          .backgroundColor($r('sys.color.ohos_id_color_emphasize'))
          .margin({ top: 12 })
          .onClick(() => { this.start(); })
        Button('停止')
          .type(ButtonType.Capsule)
          .height(40)
          .width('100%')
          .fontSize(16)
          .fontColor($r('sys.color.ohos_id_color_text_primary'))
          .backgroundColor($r('sys.color.ohos_id_color_component_normal'))
          .margin({ top: 8 })
          .onClick(() => { this.pip?.stopPiP(); })
      }
      .width('100%')
      .height('100%')
      .padding(16)
      .backgroundColor($r('sys.color.ohos_id_color_background'))
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
    Column() {
      Text('闪控窗')
        .fontSize(20)
        .fontWeight(FontWeight.Medium)
        .fontColor($r('sys.color.ohos_id_color_text_primary'))
        .width('100%')
      Text('退后台后继续显示应用页面')
        .fontSize(14)
        .fontColor($r('sys.color.ohos_id_color_text_secondary'))
        .margin({ top: 8 })
        .width('100%')
      Blank()
      Button('启动')
        .type(ButtonType.Capsule)
        .height(40)
        .width('100%')
        .fontSize(16)
        .fontColor(Color.White)
        .backgroundColor($r('sys.color.ohos_id_color_emphasize'))
        .onClick(() => { this.start(); })
    }
    .width('100%')
    .height('100%')
    .padding({ left: 16, right: 16, top: 24, bottom: 24 })
    .backgroundColor($r('sys.color.ohos_id_color_background'))
  }
}
```

`pages/FloatPanel`（须写入 `main_pages.json`）。可点控件避开 `getWindowProperties().avoidArea`。视觉见 [docs/ui-style.md](docs/ui-style.md)。

```ts
@Entry
@Component
struct FloatPanel {
  build() {
    Column() {
      Text('盯盘')
        .fontSize(16)
        .fontWeight(FontWeight.Medium)
        .fontColor($r('sys.color.ohos_id_color_text_primary'))
      Text('沪指 +0.8%')
        .fontSize(14)
        .fontColor($r('sys.color.ohos_id_color_text_secondary'))
        .margin({ top: 4 })
    }
    .width('100%')
    .height('100%')
    .padding(12)
    .justifyContent(FlexAlign.Center)
    .backgroundColor($r('sys.color.ohos_id_color_background'))
  }
}
```

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

  build() {
    Column() {
      Text('闪控球')
        .fontSize(20)
        .fontWeight(FontWeight.Medium)
        .fontColor($r('sys.color.ohos_id_color_text_primary'))
        .width('100%')
      Text('系统绘制贴边球，这里只负责启动')
        .fontSize(14)
        .fontColor($r('sys.color.ohos_id_color_text_secondary'))
        .margin({ top: 8 })
        .width('100%')
      Blank()
      Button('启动')
        .type(ButtonType.Capsule)
        .height(40)
        .width('100%')
        .fontSize(16)
        .fontColor(Color.White)
        .backgroundColor($r('sys.color.ohos_id_color_emphasize'))
        .onClick(() => { this.start(); })
    }
    .width('100%')
    .height('100%')
    .padding({ left: 16, right: 16, top: 24, bottom: 24 })
    .backgroundColor($r('sys.color.ohos_id_color_background'))
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
