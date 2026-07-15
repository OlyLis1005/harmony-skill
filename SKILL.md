---
name: harmony-skill
description: 鸿蒙（HarmonyOS NEXT）应用开发技能包。触发条件：用户需要开发鸿蒙应用、编写 ArkTS/ArkUI 代码、创建鸿蒙工程、调试、Electron 鸿蒙 PC 开发、迁移 Electron 项目、或咨询鸿蒙开发问题。覆盖 ArkTS 语法、ArkUI 声明式 UI、Stage 模型、路由导航、网络请求、数据持久化、Electron for HarmonyOS。
version: 2.2.0
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
   - Electron 调用 ArkTS 系统能力 → `systemPreferences.callArkTSFunction()`（第15章）
   - Electron fork/spawn 子进程 → HNP 打包（第16章）
   - Electron 加载 Native Addon → Mock 替代 / AKI 桥接
9. **TS/JS 代码翻译为 ArkTS** → 参照 `references/arkts-restrictions.md`
10. **应用签名/上架** → 参照「应用签名与发布」
11. **打包构建 / 多 Flavor（App/HAP）/ 签名排错** → 参照「打包构建与多 Flavor（App/HAP）」

## 工程结构

```
MyApplication/
├── AppScope/
│   ├── app.json5          # 应用级配置（包名、版本、图标）
│   └── resources/
├── entry/                 # 默认模块（HAP 包）
│   ├── src/main/
│   │   ├── ets/           # ArkTS 源码
│   │   │   ├── entryability/EntryAbility.ets
│   │   │   └── pages/Index.ets
│   │   ├── resources/
│   │   └── module.json5   # 模块配置 + 权限声明
│   └── build-profile.json5
├── hvigor/                # 构建工具
├── oh-package.json5       # 依赖管理
└── build-profile.json5    # 工程级构建配置
```

**关键配置**：
- `app.json5`：bundleName、版本号、图标、最小兼容 API 版本
- `module.json5`：模块名、Ability 注册、权限声明
- `build-profile.json5`：签名配置、构建类型

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

## Electron for HarmonyOS

> 详细参考见 `references/electron-harmonyos.md`

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

## 打包构建与多 Flavor（App/HAP）

> 适用场景：打正式 app / 测试 app（含 hap）/ 调试 hap；处理 SignHap 签名失败；统一打包工具链。

### hvigorw 任务层级（关键坑）

| 任务 | 层级 | 参数 |
|------|------|------|
| `assembleApp` | **项目级** | `--mode project` |
| `assembleHap` | **模块级** | `--mode module -p module=<模块名>@<target>`（如 `module=electron@default`） |

- `assembleHap` 必须用 `--mode module` 并指定模块，否则报 `Task ['assembleHap'] was not found in the project`。
- 典型三 flavor 命令（product 决定签名，buildMode 决定调试/发布）：
  - 正式 app：`assembleApp --mode project -p product=prod -p buildMode=release` → 产出 `.app`
  - 测试 app：`assembleApp --mode project -p product=default -p buildMode=release` → 产出 `.app` + `.hap`
  - 调试 hap：`assembleHap --mode module -p module=electron@default -p product=default -p buildMode=debug` → 产出 `.hap`
- 构建前先 `ohpm install`，再调 hvigorw。

### 多产品（product）与签名

- `build-profile.json5` 中 `app.products` 定义产品（如 `default`、`prod`），每个 product 的 `signingConfig` 指向 `app.signingConfigs` 中的某条（release / debug 签名）。
- **常见遗漏**：模块级 `modules[].targets[].applyToProducts` 必须包含该模块要构建的所有 product。新增 `prod` 等产品后，若忘了把它加入 `electron` 模块的 `applyToProducts`，则 `-p product=prod` 会因无模块可构建而失败。
- `storePassword`/`keyPassword` 在 `build-profile.json5` 中是 DevEco **加密串**（`0000001A…` 前缀），不是明文，排错时勿当明文密码处理。

### App 包 hdc 安装报 9568320（no signature file）— appWithSignedPkg

- **报错特征**：`hdc install xxx.app` → `code:9568320 error: no signature file`。
- **根因**：`assembleApp` 默认产出的 app 包里内层 hap/hsp 是**未签名**的（即便产物名叫 `*-signed.app`），设备安装时校验内层无签名 → 9568320。
- **修复**：在 `build-profile.json5` 中为 product 开启 `packOptions.appWithSignedPkg: true`，让构建额外把 app 包内的 hap/hsp 也签名。开启后普通 `*-signed.app` 即可直接 `hdc install` 成功（无需改用 `*-all-signed.app`）。DevEco Studio 6.0.2 Beta1+ 支持。
- **层级坑（重要）**：`packOptions` **不能**放顶层，也**不能**直接放 `app` 下，否则报 `00303038 Configuration Error / property name must be valid`。`app` 直接子字段仅允许 `signingConfigs / products / buildModeSet / multiProjects / capabilities`。
  - **正确层级**：`app → products[] → buildOption → packOptions → appWithSignedPkg`。每个需要构建 app 的 product（如 `default`、`prod`）都要各自加。
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
- **前置条件**：需已配置签名材料，且 `hvigor-config.json5` 的 `enableSignTask` 未设为 `false`。

### 签名 SignHap 失败：JDK 版本不匹配（高频坑）

- **报错特征**：`Init keystore failed` / `parseAlgParameters failed: ObjectIdentifier() -- data isn't an object ID (tag = 48)`。
- **根因**：本地 hvigor 守护进程用的 JDK 太旧（**JDK 8**），而 `.p12` 由 **JDK 9+** 生成。JDK 9+ 的 PKCS12 用 SHA-256 MAC + PBES2/AES-256 算法参数，JDK 8 解析不了 → 报 `tag = 48 (0x30 = ASN.1 SEQUENCE)` 错误。这对应官方建议「The keystore was created by a newer JDK version」。
- **确认**：用 `keytool -list -keystore xxx.p12 -storetype PKCS12 -storepass <pass>`，JDK 8 复现完全相同的报错，JDK 17 正常列出条目。解析 `.p12` 的 ASN.1 可见 MAC OID=`2.16.840.1.101.3.4.2.1`（SHA-256）、证书保护 OID=`1.2.840.113549.1.5.13`（PBES2）。
- **修复（推荐）**：构建前把 `JAVA_HOME` 指向 **JDK 11/17**（command-line-tools 不自带 JDK 时尤其必要），再调 hvigorw；建议在打包脚本里显式设，避免依赖当前 shell 碰巧解析到的 JDK。
- **不要**用 JDK 8 重新生成 `.p12`：已向 AppGallery 申请的 `.cer`/`.p7b` 是按当前密钥签发的，重生成会让证书链失效。
- CI 通常自带较新 JBR 所以能过；本地和 CI 不一致往往就是本地 JDK 太旧。

### 工具链约定

- 统一使用 OHOS **Command Line Tools** 的 `hvigorw`（Linux 为 shell 脚本，Windows 为 `hvigorw.bat`）和 `ohpm`（装依赖）。**不要再使用 `devecocli`**。
- Windows 上 `.bat` 需经 `cmd //c` 调用，路径用 `cygpath -w` 转换。
- PowerShell 脚本（`.ps1`）在中文代码页 Windows 必须以 **UTF-8 BOM（utf-8-sig）** 保存，否则中文变 GBK 乱码、YAML 解析失败；普通 `.sh` 注释保持 UTF-8/LF。

### 产物收集

- `assembleHap` / `assembleApp` 在 `electron/build/<target>/outputs/default/` 下产出 `*-signed.hap` 和 `*-unsigned.hap`，**收集时优先取 `*-signed.hap`**。

## 参考文档

- `references/arkts-restrictions.md`：**ArkTS 编译器严格限制与错误对照表**（TS→ArkTS 翻译必读）
- `references/arkui-components.md`：ArkUI 组件属性和事件参考
- `references/project-templates.md`：常用工程模板和代码片段
- `references/electron-harmonyos.md`：Electron for HarmonyOS 完整开发指南

**官方文档**：
- 文档中心：https://developer.huawei.com/consumer/cn/doc/
- API 参考：https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkui-ts
