---
name: harmony-skill
description: 鸿蒙（HarmonyOS NEXT）应用开发技能包。触发条件：用户需要开发鸿蒙应用、编写 ArkTS/ArkUI 代码、创建鸿蒙工程、调试、Electron 鸿蒙 PC 开发、迁移 Electron 项目、配置多 Flavor 打包/签名、打包 HNP 后端、定制应用名/图标、处理资源缓存，或咨询鸿蒙开发问题。覆盖 ArkTS 语法与严格限制、ArkUI 声明式 UI、Stage 模型、路由导航、网络请求、数据持久化、Electron for HarmonyOS 工程化。
version: 3.2.0
---

# 鸿蒙应用开发

## 概述

以华为官方文档（https://developer.huawei.com/consumer/cn/doc/）为准。

**核心约束（必须遵守）**：
1. **禁止猜测**：生成 ArkTS/ArkUI 代码时，必须严格基于本技能文档和华为官方文档中的内容。禁止凭训练数据"推测"API、编造不存在的属性/方法，或使用过时的写法。
2. **不确定时依赖技能文档**：遇到不确定的 API、组件属性、版本差异时，优先查阅本技能文档（`references/` 目录）。
3. **华为官方文档获取方式**：华为官方文档（https://developer.huawei.com/consumer/cn/doc/）和 GitCode Electron 文档（https://gitcode.com/openharmony-sig/electron）**均无法通过 WebFetch 直接获取**。若技能文档无法覆盖当前问题，**必须使用 agent-browser（浏览器自动化）打开对应页面获取信息**，绝不可凭猜测编造。
4. **冲突时以官方为准**：如果技能文档与官方文档存在冲突，以官方文档为准，并告知用户差异。
5. **明确告知不确定**：如果通过 agent-browser 也无法获取，**必须明确告知用户"无法确认"**，绝不可编造一个答案。
6. **写完代码后主动编译验证**：生成或修改 ArkTS/ArkUI 代码后，**必须立即尝试编译**（`hvigorw assembleHap` 或项目中的构建命令），通过编译错误来发现和修复问题，而不是等用户去编译。如果编译环境不可用，则需明确告知用户"无法自动编译验证，请手动编译确认"。

## 触发条件与决策流

用户请求涉及以下任一场景时触发：

1. **从零创建鸿蒙应用** → 参照「工程结构」生成脚手架
2. **编写 UI 页面** → 使用 ArkUI 声明式语法，参照「ArkUI 核心速查」
3. **状态管理问题** → 查阅「装饰器速查」
4. **页面跳转/导航** → 简单场景用 Router，复杂场景用 Navigation
5. **网络请求** → 参照「网络请求」模板
6. **数据存储** → 轻量用 Preferences，结构化数据用 RelationalStore
7. **调用系统能力** → 查阅「常用系统能力」表，优先使用技能文档中的内容
8. **Electron 开发鸿蒙 PC 应用 / 迁移** → 参照「Electron for HarmonyOS」
   - 从零创建 → 环境搭建 + 第一个应用
   - 迁移现有项目 → 迁移指南 + 架构差异
   - Electron 调用 ArkTS 系统能力 → `systemPreferences.callArkTSFunction()`（references）
   - Electron fork/spawn 子进程 → HNP 打包（见「Electron for HarmonyOS / HNP 后端打包」）
   - Electron 加载 Native Addon → Mock 替代 / AKI 桥接
9. **TS/JS 代码翻译为 ArkTS** → 参照 `references/arkts-restrictions.md`
10. **应用签名/上架** → 参照「打包构建与多 Flavor（App/HAP）」的签名小节
11. **打包构建 / 多 Flavor（App/HAP）/ 签名排错** → 参照「打包构建与多 Flavor（App/HAP）」
12. **定制应用名 / 桌面图标 / 资源不刷新** → 参照「Electron for HarmonyOS」应用名图标与资源缓存小节

## 工程结构

```
MyApplication/
├── AppScope/
│   ├── app.json5          # 应用级配置（包名、版本、图标）
│   └── resources/
├── entry/                 # 默认模块（HAP 包）；Electron 工程则用 electron 模块
│   ├── src/main/
│   │   ├── ets/           # ArkTS 源码
│   │   │   ├── entryability/EntryAbility.ets
│   │   │   └── pages/Index.ets
│   │   ├── resources/
│   │   └── module.json5   # 模块配置 + 权限声明 + 应用名/图标来源
│   └── build-profile.json5
├── hvigor/                # 构建工具
├── oh-package.json5       # 依赖管理
└── build-profile.json5    # 工程级构建配置（signingConfigs / products / buildModeSet）
```

**关键配置**：
- `app.json5`：bundleName、版本号、图标、最小兼容 API 版本
- `module.json5`：模块名、Ability 注册、权限声明、应用名/图标（**单 EntryAbility 应用的桌面名/图标来自这里，不是 AppScope**）
- `build-profile.json5`：签名配置、构建类型、多产品

> **Electron for HarmonyOS 工程结构差异**：模块不是 `entry`，而是 `electron`（桌面壳 + `EntryAbility`）、`web_engine`（Electron 运行时 + 内置 web 应用）、可选 HNP 后端。详见「Electron for HarmonyOS」。

## ArkTS 核心限制速查（最常见的 7 个坑）

| 限制 | 错误写法 | ArkTS 替代 |
|------|---------|-----------|
| 禁止对象字面量类型 | `{ x: number }` 做类型 | 先定义 `interface` |
| 禁止无类型对象 | `const x = { a: 1 }` | 加类型注解 `const x: T = { ... }` |
| 禁止 spread | `{ ...obj }` / `[...arr]` | 逐字段拷贝 / clone 函数 |
| 禁止 any/unknown | `let x: any` | 明确 interface 类型；JSON 反序列化用 `Object` |
| 禁止 `as` 断言 | `'val' as Type` | `const x: Type = 'val'` |
| 禁止索引访问 | `obj[key]` | if/switch 展开 / getter/setter |
| 禁止可选属性 | `field?: Type` | 必填 + 默认值 |

> **TS → ArkTS 完整翻译表见 `references/arkts-restrictions.md`**（含编译错误码、错误示例、正确写法）

## 装饰器速查

| 装饰器 | 作用 | 数据流向 |
|--------|------|----------|
| `@Entry` | 页面入口组件 | — |
| `@Component` | 自定义组件 | — |
| `@State` | 组件内部可变状态（观察第一层） | 本组件内 |
| `@Prop` | 父→子单向数据传递 | 父→子 |
| `@Link` | 父子双向数据绑定 | 父↔子 |
| `@Provide` | 跨层级向下提供数据 | 祖先→后代 |
| `@Consume` | 跨层级向上消费数据 | 后代←祖先 |
| `@Observed` | 标记可深层观察的 class | — |
| `@ObjectLink` | 子组件接收 @Observed 实例 | 双向 |
| `@Watch` | 监听状态变化回调 | — |
| `@Builder` | 轻量 UI 复用函数 | — |
| `@Extend` | 扩展原生组件样式 | — |
| `@Styles` | 样式复用 | — |

## 弹窗选用指南

| 场景 | 推荐组件 |
|------|---------|
| 简单确认/提示 | `this.getUIContext().showAlertDialog()` |
| 列表选择 | `this.getUIContext().showActionSheet()` |
| **自定义样式** | `@CustomDialog` + `CustomDialogController` |
| 轻量提示 | `this.promptAction.showToast()`（需先创建 `PromptAction` 实例，详见 `references/arkts-restrictions.md` 第 17 条） |

> ⚠️ `AlertDialog.show()` / `ActionSheet.show()` 已废弃，必须用 `getUIContext()` 方式。
> **CustomDialog 标准模式见 `references/arkts-restrictions.md` 第 17 条。**

## 核心代码模板

### 页面基本结构
```typescript
@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS'

  build() {
    Column() {
      Text(this.message).fontSize(30).fontWeight(FontWeight.Bold)
      Button('点击').onClick(() => { this.message = '你好鸿蒙' })
    }
    .width('100%').height('100%').justifyContent(FlexAlign.Center)
  }
}
```

### Router 路由
```typescript
import router from '@ohos.router'

// 跳转
router.pushUrl({ url: 'pages/DetailPage', params: { id: 1 } })
// 返回
router.back()
// 获取参数
const params = router.getParams() as Object;
// 注意：ArkTS 不支持 Record<K, V>，建议定义 interface 替代
```

### Navigation（推荐）
```typescript
@Entry
@Component
struct MainPage {
  navStack: NavPathStack = new NavPathStack()
  build() {
    Navigation(this.navStack) {
      Button('跳转').onClick(() => {
        this.navStack.pushPath({ name: 'DetailPage', param: { id: 1 } })
      })
    }
  }
}
```

### 网络请求
```typescript
import http from '@ohos.net.http'

async function fetchData(url: string): Promise<string> {
  const req = http.createHttp()
  const res = await req.request(url, {
    method: http.RequestMethod.GET,
    header: { 'Content-Type': 'application/json' },
    connectTimeout: 60000, readTimeout: 60000,
  })
  req.destroy()
  if (res.responseCode === 200) return res.result as string
  throw new Error(`HTTP ${res.responseCode}`)
}
```

### 权限声明（module.json5）
```json
{
  "module": {
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" }
    ]
  }
}
```

### Preferences 存储
```typescript
import preferences from '@ohos.data.preferences'

const pref = await preferences.getPreferences(context, 'my_store')
await pref.put('key', 'value')
await pref.flush()
const value = await pref.get('key', 'default')
```

## UIAbility 生命周期

```
onCreate → onWindowStageCreate → onForeground → onBackground → onWindowStageDestroy → onDestroy
```

| 模式 | 说明 |
|------|------|
| singleton | 单实例（默认），所有请求共享同一实例 |
| specified | 指定实例，通过 key 决定是否复用 |
| multiton | 多实例，每次启动创建新实例 |

## 常用系统能力

| 能力 | 模块 |
|------|------|
| 网络请求 | `@ohos.net.http` |
| 数据存储 | `@ohos.data.preferences` / `@ohos.data.relationalStore` |
| 文件管理 | `@ohos.file.fs` |
| 相机 | `@ohos.multimedia.camera` |
| 图片 | `@ohos.multimedia.image` |
| 媒体 | `@ohos.multimedia.media` |
| 位置 | `@ohos.geoLocationManager` |
| 通知 | `@ohos.notificationManager` |
| 权限 | `@ohos.abilityAccessCtrl` |
| 设备信息 | `@ohos.deviceInfo` |
| 窗口 | `@ohos.window` |
| 应用上下文 | `@ohos.app.ability.common` |

## Electron for HarmonyOS

> 详细参考见 `references/electron-harmonyos.md`（开发/迁移指南、HNP 后端打包 §16、应用名/图标定制 §17）

**核心差异**：鸿蒙 Electron 不支持 Node.js 运行时，原生能力需通过鸿蒙系统 API 实现。

```javascript
// 必需配置
app.disableHardwareAcceleration()
app.commandLine.appendSwitch('disable-gpu')

new BrowserWindow({
  webPreferences: {
    contextIsolation: true,   // 必须开启
    nodeIntegration: false,  // 必须关闭
  }
})
```

### 工程架构（本项目实际形态）

单 EntryAbility 的 Electron 鸿蒙应用由以下模块组成（以 `build-profile.json5` 的 `modules[]` 为准）：

- **electron 模块**：桌面应用外壳，承载 `EntryAbility`（启动器与浏览器能力）。**桌面显示的应用名与图标由该模块的 `module.json5` 决定**（见下节），不是 AppScope。
- **web_engine 模块**：承载 Electron 运行时与 renderer，内置 web 应用放在 `web_engine/src/main/resources/resfile/resources/app`（由前端构建产物同步进来）。运行期通过 KVStore 把 web 应用的 label/icon 写入 Ability 实例信息，但**仅影响运行时窗口标题，不影响桌面图标**。
- **HNP 后端（可选）**：把 Python 或原生 ELF 后端打包为 `.hnp`，在 `module.json5` 的 `hnpPackages` 声明，前端用 `child_process` 调起。详见「HNP 后端打包」。

> `deviceTypes` 按目标设备填写（如 `2in1`）；跨设备类型时确保资源与能力兼容。

### 应用名与桌面图标定制（单 EntryAbility 应用，已实测验证）

> 本项目踩坑结论：官方 README「替换图标在 AppScope media」对**启动器图标**是误导。桌面元数据走 **Ability 级**。

**应用显示名（桌面 / 任务里的名字）**
- 来源：`EntryAbility.label` → 字符串资源项 **`EntryAbility_label`**。
- 实际位置：`electron/src/main/resources/zh_CN/element/string.json`（中文系统走 `zh_CN`，覆盖 `base`）；英文环境改 `en_US`。**只改 `base` 不够**，各语言目录须改一致。
- AppScope 的 `app_name` **不影响桌面显示名**，仅用于应用信息页。

**桌面启动器图标**
- 来源：`EntryAbility.icon` → `$media:app_icon` → 文件 **`electron/src/main/resources/base/media/app_icon.png`**。
- 改这一张即可（electron 模块通常只有 `base/media`，无语言覆盖）；同目录 `startIcon.png`（启动窗口 / 任务列表图标）一并换，保持一致。
- AppScope 的 `layered_image`（`foreground.png` + `background.png`）只用于应用信息页 / 兜底，**不改桌面图标**。
- `layered_image` 为分层合成图标（前景盖背景 + 系统遮罩），若需整图显示，确保 `foreground.png` 主体居中、四周留透明，并调好 `background.png` 底色。

### 资源缓存与设备刷新

- **改资源后桌面仍显示旧值**：先 `hvigorw clean` 清增量缓存再构建（强制重编资源表 `ResourceTable`/`index`）。打包脚本可默认 clean，用环境变量（如 `PACK_NO_CLEAN=1`）跳过以保留增量速度；`clean` 只删 `build/`，不影响源码树里由同步脚本放入模块目录的产物（如 `hnp/`）。
- **图标按包名缓存**：改图标后 `hdc install` 覆盖升级 → **应用名立即刷新，但图标位图被系统按 bundleName 缓存、升级安装不刷新**，出现"名字变了、图标没变"。解决：`hdc uninstall <bundleName>` 后重新 `hdc install`；仍不刷新则重启设备一次。

### HNP 后端打包（Python / 原生 ELF 运行于鸿蒙）

> 适用：把后端（Python 解释器 + 业务代码，或任意 ELF）以 HNP 形式随应用分发，由前端 `child_process` 调起，进程隔离、不依赖 Node 运行时版本。

**打包 .hnp**：用 `hnpcli pack -i <待打包目录> -o <输出目录>`（输出目录不能在待打包目录内，否则递归）。待打包目录放 ELF + `hnp.json`：
```json
{ "type": "hnp-config", "name": "backend", "version": "1.0",
  "install": { "links": [ { "source": "bin/backend", "target": "backend" } ] } }
```

**接入工程**：把 `.hnp` 放到工程根 `hnp/<ABI>/`（如 `hnp/arm64-v8a/`）；`module.json5` 加 `hnpPackages: [ { "package": "backend.hnp", "type": "private" } ]`；前端用 `child_process.spawn` 执行，**路径用沙箱物理路径**：private 类型 → `data/app/backend.org/backend_1.0/bin/backend`。

**关键补丁：让 command-line-tools 把 hnp 打进 hap**（默认 hap 打包脚本不含 `--hnp-path`，否则运行时 `child_process` 找不到二进制）：
- 改的是**已安装的工具链文件**（非工程文件；CI / 他人环境都要各自打）。
- `packing-tool-options.js`（`PackingToolOptions` 类）增加 `addHnpPath(t) { return this.addFieldAndPath("--hnp-path", t); }`。
- `base-pack-hap-task.js` 的 `generateCommand` 中，在 `new PackingToolOptions()` 之后追加：若 `process.cwd()` 下存在 `hnp` 目录，则 `a.addHnpPath(hnpPath)`（变量名随 SDK 版本变化，有的版本是 `a`、有的是 `s`，按实际文件调整；已打过补丁则跳过）。DevEco Studio 与 command-line-tools 同 SDK，插件目录布局一致（通常在 `tools/hvigor/hvigor-ohos-plugin/src/...`）。

## 打包构建与多 Flavor（App/HAP）

> 适用场景：打正式 app / 测试 app（含 hap）/ 调试 hap；处理 SignHap 签名失败；统一打包工具链。

### 构建工具链约定

- 统一使用 OHOS **Command Line Tools** 的 `hvigorw`（Linux 为 shell 脚本，Windows 为 `hvigorw.bat`）和 `ohpm`（装依赖）。**弃用 `devecocli`**。
- **JDK**：构建/签名必须 **JDK 17**（详见 SignHap 小节）。command-line-tools 不自带 JDK 时需显式 `export JAVA_HOME`。
- **Node**：Node 22（hvigorw 是 Node 启动器）。
- **Windows 调用**：`.bat` 须经 `cmd //c` 调用，路径用 `cygpath -w` 转 Windows 格式；`.ps1` 须 **UTF-8 BOM** 保存。
- **跨环境探测（不写死机器路径）**：用 `COMMAND_LINE_TOOLS` / `OHOS_SDK_HOME` 定位工具根；候选常见路径（如 `$HOME/DevEco/command-line-tools`、`/opt/harmonyos/command-line-tools`）；最后 `command -v` 兜底。同一套 `detect_hvigorw` / `detect_ohpm` / `detect_hnpcli` 逻辑在 Win/Linux 下自动加 `.bat` 后缀。
- **标准构建顺序**：`ohpm install` →（资源变更时 `hvigorw clean`）→ `hvigorw <任务>`。

### hvigorw 任务层级（关键坑）

| 任务 | 层级 | 参数 |
|------|------|------|
| `assembleApp` | **项目级** | `--mode project` |
| `assembleHap` | **模块级** | `--mode module -p module=<模块名>@<target>`（如 `module=electron@default`） |

- `assembleHap` 必须用 `--mode module` 并指定模块，否则报 `Task ['assembleHap'] was not found in the project`（混用 mode 必挂）。
- 三 flavor 命令（product 决定签名，buildMode 决定调试/发布）：
  - **prod（正式 app）**：`assembleApp --mode project -p product=prod -p buildMode=release` → 仅 `.app`
  - **test（测试 app）**：`assembleApp --mode project -p product=default -p buildMode=release` → `.app` + `.hap`
  - **debug（调试 hap）**：`assembleHap --mode module -p module=electron@default -p product=default -p buildMode=debug` → 仅 `.hap`

### 多产品（product）与签名

- `build-profile.json5` 中 `app.products` 定义产品（如 `default`、`prod`），每个 product 的 `signingConfig` 指向 `app.signingConfigs` 中的某条（release / debug 签名）。
- **常见遗漏**：模块级 `modules[].targets[].applyToProducts` 必须包含该模块要构建的所有 product。新增 `prod` 等产品后，若忘了把它加入 `electron` 模块的 `applyToProducts`，则 `-p product=prod` 会因无模块可构建而失败。
- `storePassword`/`keyPassword` 在 `build-profile.json5` 中是 DevEco **加密串**（`0000001A…` 前缀），不是明文，排错时勿当明文密码处理。

### App 包 hdc 安装报错码（9568320 / 9568448）

- `9568320 error: no signature file`：**根因** `assembleApp` 默认产出的 app 包内层 hap/hsp 未签名 → 开启 `packOptions.appWithSignedPkg: true`（见下）让内层也签名，普通 `*-signed.app` 即可 `hdc install` 成功。
- `9568448 verify app signature failed`：App Pack（`.app`）外层无签名或设备无对应发布证书。本地真机调试**应装 `.hap` 而非 `.app`**；`.app` 面向上架/分发。

### 9568320 修复 — appWithSignedPkg（层级坑重要）

- **正确层级**：`app → products[] → buildOption → packOptions → appWithSignedPkg`，**每个要构建 app 的 product（如 `default`、`prod`）都要各自加**。
  ```json5
  // build-profile.json5
  "products": [
    {
      "name": "default",
      "signingConfig": "debug",
      "buildOption": {
        "packOptions": { "appWithSignedPkg": true }
      }
    }
  ]
  ```
- **层级雷区（报 `00303038 Configuration Error`）**：
  - 不能放**顶层**（顶层仅允许 `app` / `modules`）；
  - 不能放 `app` **直接子级**（`app` 直接子字段仅 `signingConfigs / products / buildModeSet / multiProjects / capabilities`）。
  - 前置：签名材料已配、`hvigor-config.json5` 的 `enableSignTask` 未设为 `false`。

### 签名 SignHap 失败：JDK 版本不匹配（高频坑）

- **报错特征**：`Init keystore failed` / `parseAlgParameters failed: ObjectIdentifier() -- data isn't an object ID (tag = 48)`。
- **根因**：本地 hvigor 守护进程用的 JDK 太旧（**JDK 8**），而 `.p12` 由 **JDK 9+** 生成。JDK 9+ 的 PKCS12 用 SHA-256 MAC + PBES2/AES-256 算法参数，JDK 8 解析不了 → 报 `tag = 48 (0x30 = ASN.1 SEQUENCE)` 错误。这对应官方建议「The keystore was created by a newer JDK version」。
- **确认**：用 `keytool -list -keystore xxx.p12 -storetype PKCS12 -storepass <pass>`，JDK 8 复现完全相同的报错，JDK 17 正常列出条目。
- **修复（推荐）**：构建前把 `JAVA_HOME` 指向 **JDK 11/17**（command-line-tools 不自带 JDK 时尤其必要），再调 hvigorw；建议在打包脚本里显式设，避免依赖当前 shell 碰巧解析到的 JDK。
- **不要**用 JDK 8 重新生成 `.p12`：已向 AppGallery 申请的 `.cer`/`.p7b` 是按当前密钥签发的，重生成会让证书链失效。
- CI 通常自带较新 JBR 所以能过；本地和 CI 不一致往往就是本地 JDK 太旧。

### 产物收集

- `assembleHap` / `assembleApp` 在 `electron/build/<target>/outputs/default/` 下产出 `*-signed.hap` 和 `*-unsigned.hap`，**收集时优先取 `*-signed.hap`**；app 同理优先 `*-signed.app`（排除 `*-unsigned`，选错 unsigned 会直接安装失败）。

## 参考文档

- `references/arkts-restrictions.md`：**ArkTS 编译器严格限制与错误对照表**（TS→ArkTS 翻译必读）
- `references/arkui-components.md`：ArkUI 组件属性和事件参考
- `references/project-templates.md`：常用工程模板和代码片段
- `references/electron-harmonyos.md`：Electron for HarmonyOS 完整开发指南（环境、架构、生命周期、HNP 后端打包 §16、应用名/图标定制 §17）

**官方文档**：
- 文档中心：https://developer.huawei.com/consumer/cn/doc/
- API 参考：https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkui-ts
