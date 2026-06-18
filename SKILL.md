---
name: harmony-skill
description: 鸿蒙（HarmonyOS NEXT）应用开发技能包。触发条件：用户需要开发鸿蒙应用、编写 ArkTS/ArkUI 代码、创建鸿蒙工程、调试、Electron 鸿蒙 PC 开发、迁移 Electron 项目、或咨询鸿蒙开发问题。覆盖 ArkTS 语法、ArkUI 声明式 UI、Stage 模型、路由导航、网络请求、数据持久化、Electron for HarmonyOS。
version: 2.0.0
---

# 鸿蒙应用开发

## 概述

以华为官方文档（https://developer.huawei.com/consumer/cn/doc/）为准。遇到 API 细节优先 WebFetch 官方文档获取最新信息。

## 触发条件与决策流

用户请求涉及以下任一场景时触发：

1. **从零创建鸿蒙应用** → 参照「工程结构」生成脚手架
2. **编写 UI 页面** → 使用 ArkUI 声明式语法，参照「ArkUI 核心速查」
3. **状态管理问题** → 查阅「装饰器速查」
4. **页面跳转/导航** → 简单场景用 Router，复杂场景用 Navigation
5. **网络请求** → 参照「网络请求」模板
6. **数据存储** → 轻量用 Preferences，结构化数据用 RelationalStore
7. **调用系统能力** → 查阅「常用系统能力」表，必要时 WebFetch 官方 API
8. **Electron 开发鸿蒙 PC 应用 / 迁移** → 参照「Electron for HarmonyOS」
   - 从零创建 → 环境搭建 + 第一个应用
   - 迁移现有项目 → 迁移指南 + 架构差异
   - Electron 调用 ArkTS 系统能力 → `systemPreferences.callArkTSFunction()`（第15章）
   - Electron fork/spawn 子进程 → HNP 打包（第16章）
   - Electron 加载 Native Addon → Mock 替代 / AKI 桥接
9. **TS/JS 代码翻译为 ArkTS** → 参照 `references/arkts-restrictions.md`
10. **应用签名/上架** → 参照「应用签名与发布」

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

## 参考文档

- `references/arkts-restrictions.md`：**ArkTS 编译器严格限制与错误对照表**（TS→ArkTS 翻译必读）
- `references/arkui-components.md`：ArkUI 组件属性和事件参考
- `references/project-templates.md`：常用工程模板和代码片段
- `references/electron-harmonyos.md`：Electron for HarmonyOS 完整开发指南

**官方文档**：
- 文档中心：https://developer.huawei.com/consumer/cn/doc/
- API 参考：https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkui-ts
