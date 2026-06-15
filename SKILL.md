---
name: harmony-skill
description: 鸿蒙（HarmonyOS NEXT）应用开发技能包。当用户需要开发鸿蒙应用、编写 ArkTS/ArkUI 代码、创建鸿蒙工程、调试鸿蒙应用、使用 Electron 框架开发鸿蒙 PC 应用、将现有 Electron 项目迁移到鸿蒙、或咨询鸿蒙开发相关问题时触发此技能。覆盖环境搭建、工程结构、ArkTS 语法、ArkUI 声明式 UI、Electron for HarmonyOS、应用模型（Stage 模型）、路由导航、网络请求、数据持久化、应用签名与发布等全流程开发场景。以华为官方文档为准。
version: 1.5.1
agent_created: true
---

# 鸿蒙应用开发

## 版本记录

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0.0 | 2026-06-10 | 初始版本：ArkTS 限制速查、ArkUI 装饰器、工程结构、Electron 鸿蒙支持 |
| 1.0.1 | 2026-06-11 | 新增第 12-13 条：@State/@Observed/@ObjectLink 深层观察机制 |
| 1.1.0 | 2026-06-12 | 新增第 14-15 条：`as` 断言禁止(10605008)、CustomComponent 属性名冲突；速查表+翻译表新增对应行 |
| 1.2.0 | 2026-06-12 | 新增第 16 条：AlertDialog.show/ActionSheet.show 废弃→UIContext 方法；速查表新增对应行 |
| 1.3.0 | 2026-06-12 | ActionSheet 样式陷阱 + CustomDialog 标准模式 + 弹窗最佳实践章节 |
| 1.3.1 | 2026-06-12 | **⚠️ 修正 CustomDialog controller 自引用 bug**：自引用 `= new CustomDialogController({ builder: 自身 })` 导致 controller undefined |
| 1.4.0 | 2026-06-12 | **controller 声明改为可选**（`controller?: CustomDialogController`）避免 "runtime-independent default value" 编译器错误；`close()` 改为安全调用 `?.close()` |
| 1.5.0 | 2026-06-12 | **CustomDialog 关闭改为 onClose 回调**：避免 `controller: ctrl` TDZ 错误 + `const ctrl` 需显式类型注解 + 箭头函数需用函数体语法 `{}` |
| 1.5.1 | 2026-06-15 | **`unknown` → `Object` 替代**：ArkTS 禁止 `unknown` 参数类型，JSON 反序列化时用 `Object` 接收，`JSON.parse()` 返回值必须显式 `as Object` |

## 概述

本技能用于指导 HarmonyOS NEXT 应用开发全流程。以华为官方文档（https://developer.huawei.com/consumer/cn/doc/）为权威来源，覆盖从环境搭建到应用上架的完整开发链路。同时包含 Electron for HarmonyOS 支持，帮助 Web 技术栈开发者快速构建鸿蒙 PC 应用或迁移现有 Electron 项目。

## 官方文档入口

| 类别 | URL |
|------|-----|
| 文档中心 | https://developer.huawei.com/consumer/cn/doc/ |
| 应用开发概述 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-dev-overview |
| API 参考 | https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkui-ts |
| DevEco Studio | https://developer.huawei.com/consumer/cn/deveco-studio/ |
| Codelabs 实战 | https://developer.huawei.com/consumer/cn/codelabs/ |
| Electron 鸿蒙源码 | https://gitcode.com/openharmony-sig/electron |
| Electron 官方文档 | https://www.electronjs.org/docs/latest |
| Electron 社区讨论 | https://developer.huawei.com/consumer/cn/forum/block/electron |

**重要原则**：遇到 API 细节、组件属性、版本差异等问题时，优先通过 WebFetch 访问上述官方文档获取最新权威信息，而非依赖训练数据中的过时内容。

## 开发环境搭建

### 系统要求

- Windows 10/11 64位 或 macOS 12 及以上
- 内存 16GB 及以上
- 磁盘空间 100GB 以上

### 安装步骤

1. 下载安装 DevEco Studio：https://developer.huawei.com/consumer/cn/deveco-studio/
2. 首次启动完成初始化向导（同意协议、配置 Node.js、下载 SDK）
3. 配置 HarmonyOS SDK（Settings > HarmonyOS SDK）
4. 注册华为开发者账号并完成实名认证
5. 配置模拟器或连接真机调试

### 创建工程

```
DevEco Studio → File → New → Create Project
→ 选择模板（如 Empty Ability）
→ 填写项目名、包名（com.example.xxx）、SDK 版本
→ Finish
```

## 工程结构

```
MyApplication/
├── AppScope/                        # 应用全局配置
│   ├── app.json5                    # 应用级配置（包名、版本、图标等）
│   └── resources/                   # 全局资源
├── entry/                           # 默认模块（HAP 包）
│   ├── src/main/
│   │   ├── ets/                     # ArkTS 源码目录
│   │   │   ├── entryability/
│   │   │   │   └── EntryAbility.ets # 入口 Ability
│   │   │   └── pages/
│   │   │       └── Index.ets        # 首页
│   │   ├── resources/               # 模块资源
│   │   └── module.json5             # 模块配置
│   └── build-profile.json5          # 构建配置
├── hvigor/                          # 构建工具
├── oh-package.json5                 # 依赖管理
└── build-profile.json5              # 工程级构建配置
```

### 关键配置文件

- `app.json5`：应用包名（bundleName）、版本号、图标、最小兼容 API 版本
- `module.json5`：模块名、Ability 注册、权限声明、路由表
- `build-profile.json5`：签名配置、构建类型

## ArkTS 语言核心

ArkTS 基于 TypeScript 扩展，是 HarmonyOS NEXT 的主要开发语言。**ArkTS 是 TypeScript 的严格子集**，禁止了大量 TS/JS 常见写法。

> ⚠️ **TS→ArkTS 翻译必读**：ArkTS 编译器有多项严格限制，直接复制 TS 代码大概率编译失败。详见 `references/arkts-restrictions.md`。

### 核心限制速查（最常见的 7 个坑）

| 限制 | TS 写法 | ArkTS 替代 |
|------|---------|-----------|
| 禁止对象字面量类型 | `{ x: number }` 做类型 | 先定义 `interface` |
| 禁止无类型对象 | `const x = { a: 1 }` | 加类型注解 `const x: T = { a: 1 }` |
| 禁止 spread | `{ ...obj }` / `[...arr]` | 逐字段拷贝 / clone 函数 |
| 禁止 any/unknown | `let x: any` | 明确 interface 类型 |
| 禁止 `as` 断言 | `'val' as Type` | `const x: Type = 'val'` |
| 禁止索引访问 | `obj[key]` | if/switch 展开 |
| 禁止 Record<K,V> | `Record<string, T>` | 数组 + 查找函数 |
| 禁止可选属性 | `field?: Type` | 必填 + 默认值 |
| 必须显式导入类型 | 漏导 type → `Cannot find name` | import 中加全所有类型 |
| 嵌套对象属性变化不刷新 | `this.state.room.width = x` | `@Observed` class + 子组件 `@ObjectLink` |
| interface 需深层观察 | `interface X { ... }` 不触发刷新 | 改为 `@Observed class X` + constructor |
| 组件属性名与基类冲突 | `private scale: number = 1` | 改名 `drawScale` 等不冲突词 |
| AlertDialog/ActionSheet 废弃 | `AlertDialog.show()` / `ActionSheet.show()` | `this.getUIContext().showAlertDialog()` / `.showActionSheet()` |

### 基本语法要点

- 使用严格类型检查，禁止 any 类型
- 不支持 any/unknown，需显式类型注解
- 状态变量必须类型声明，禁止省略
- 使用 `@Observed` 和 `@ObjectLink` 管理嵌套对象
- **禁止内联对象字面量做类型**，必须先定义 interface/class
- **禁止 spread 操作符**，必须逐字段拷贝或写 clone 函数
- **禁止索引访问** (`obj[key]`)，必须用 if/switch 或 getter/setter

### 装饰器速查

| 装饰器 | 作用 | 数据流向 |
|--------|------|----------|
| `@Entry` | 标记页面入口组件 | — |
| `@Component` | 标记自定义组件 | — |
| `@State` | 组件内部可变状态（观察第一层） | 本组件内 |
| `@Prop` | 父→子单向数据传递 | 父→子 |
| `@Link` | 父子双向数据绑定 | 父↔子 |
| `@Provide` | 跨层级向下提供数据 | 祖先→后代 |
| `@Consume` | 跨层级向上消费数据 | 后代←祖先 |
| `@Observed` | 标记可深层观察的 class（嵌套对象属性变化） | — |
| `@ObjectLink` | 子组件接收 @Observed 实例（观察属性变化） | 双向 |
| `@Watch` | 监听状态变化回调 | — |
| `@Builder` | 轻量 UI 复用函数 | — |
| `@Extend` | 扩展原生组件样式 | — |
| `@Styles` | 样式复用 | — |

### 组件基本结构

```typescript
@Entry
@Component
struct Index {
  @State message: string = 'Hello HarmonyOS'

  build() {
    Column() {
      Text(this.message)
        .fontSize(30)
        .fontWeight(FontWeight.Bold)
      Button('点击')
        .onClick(() => {
          this.message = '你好鸿蒙'
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

## ArkUI 声明式 UI

### 布局组件

| 布局 | 用途 |
|------|------|
| `Column` | 垂直线性布局 |
| `Row` | 水平线性布局 |
| `Flex` | 弹性布局 |
| `Stack` | 层叠布局 |
| `Grid` | 网格布局 |
| `List` | 列表（支持滚动、懒加载） |
| `RelativeContainer` | 相对布局 |
| `Navigation` | 导航容器（单页/分栏模式） |
| `TabContent` + `Tabs` | 标签页 |

### 常用基础组件

```typescript
// 文本
Text('内容').fontSize(16).fontColor('#333').maxLines(1).textOverflow({ overflow: TextOverflow.Ellipsis })

// 图片
Image($r('app.media.icon')).width(100).height(100).objectFit(ImageFit.Cover)

// 按钮
Button('确认').type(ButtonType.Capsule).backgroundColor('#007DFF').onClick(() => {})

// 输入框
TextInput({ placeholder: '请输入' }).type(InputType.Normal).onChange((val) => {})

// 滚动容器
Scroll() {
  Column() { /* 子组件 */ }
}.scrollable(ScrollDirection.Vertical)

// 列表
List() {
  ForEach(this.dataList, (item: DataModel) => {
    ListItem() {
      Text(item.name)
    }
  }, (item: DataModel) => item.id.toString())
}

// 对话框（推荐 CustomDialog 自定义，或 AlertDialog 简单确认）
// ⚠️ AlertDialog.show() 已废弃，请使用 this.getUIContext().showAlertDialog()
this.getUIContext().showAlertDialog({
  title: '提示',
  message: '确认操作？',
  buttons: [
    { value: '取消', action: () => {} },
    { value: '确认', action: () => {} }
  ]
})
```

### 渲染控制

```typescript
// 条件渲染
if (this.isLoading) {
  LoadingProgress().color('#007DFF')
} else {
  Text('加载完成')
}

// 循环渲染
ForEach(this.list, (item: Item) => {
  Text(item.name)
}, (item: Item) => item.id)

// 懒加载（大数据量）
LazyForEach(this.dataSource, (item: Item) => {
  ListItem() { Text(item.name) }
}, (item: Item) => item.id)
```

## 应用模型（Stage 模型）

### 层级结构

```
Application（应用）
└── Module（模块/HAP）
    └── UIAbility（界面能力）
        └── WindowStage（窗口舞台）
            └── Page（页面）
```

### UIAbility 生命周期

```
onCreate → onWindowStageCreate → onForeground → onBackground → onWindowStageDestroy → onDestroy
```

- `onCreate`：Ability 创建时回调
- `onWindowStageCreate`：窗口舞台创建完成
- `onForeground`：切换到前台
- `onBackground`：切换到后台
- `onWindowStageDestroy`：窗口舞台销毁
- `onDestroy`：Ability 销毁

### UIAbility 启动模式

| 模式 | 说明 |
|------|------|
| singleton | 单实例（默认），所有请求共享同一实例 |
| specified | 指定实例，通过 key 决定是否复用 |
| multiton | 多实例，每次启动创建新实例 |

## 页面路由与导航

### Router（轻量路由）

```typescript
import router from '@ohos.router'

// 跳转
router.pushUrl({ url: 'pages/DetailPage', params: { id: 1 } })

// 替换当前页
router.replaceUrl({ url: 'pages/LoginPage' })

// 返回
router.back()

// 获取参数
const params = router.getParams() as Record<string, Object>
```

### Navigation（推荐，支持更复杂场景）

```typescript
// 支持导航栈管理、转场动画、深链接
@Entry
@Component
struct MainPage {
  navStack: NavPathStack = new NavPathStack()

  build() {
    Navigation(this.navStack) {
      Column() {
        Button('跳转详情').onClick(() => {
          this.navStack.pushPath({ name: 'DetailPage', param: { id: 1 } })
        })
      }
    }
  }
}
```

## 网络请求

```typescript
import http from '@ohos.net.http'

// GET 请求
async function fetchData(url: string): Promise<string> {
  const httpRequest = http.createHttp()
  const response = await httpRequest.request(url, {
    method: http.RequestMethod.GET,
    header: { 'Content-Type': 'application/json' },
    connectTimeout: 60000,
    readTimeout: 60000,
  })
  httpRequest.destroy()
  if (response.responseCode === 200) {
    return response.result as string
  }
  throw new Error(`HTTP ${response.responseCode}`)
}
```

### 权限声明

在 `module.json5` 中声明网络权限：
```json
{
  "module": {
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" }
    ]
  }
}
```

## 数据持久化

### Preferences（轻量键值存储）

```typescript
import preferences from '@ohos.data.preferences'

// 获取 Preferences 实例
const pref = await preferences.getPreferences(context, 'my_store')

// 写入
await pref.put('key', 'value')
await pref.flush()

// 读取
const value = await pref.get('key', 'default')

// 删除
await pref.delete('key')
await pref.flush()
```

### 关系型数据库（RelationalStore）

```typescript
import relationalStore from '@ohos.data.relationalStore'

// 创建/打开数据库
const store = await relationalStore.getRdbStore(context, {
  name: 'my.db',
  securityLevel: relationalStore.SecurityLevel.S1,
})

// 建表
await store.executeSql('CREATE TABLE IF NOT EXISTS user (id INTEGER PRIMARY KEY, name TEXT)')

// 插入
await store.insert('user', { id: 1, name: '张三' })

// 查询
const resultSet = await store.querySql('SELECT * FROM user')
```

## 模块化开发

| 类型 | 说明 | 使用场景 |
|------|------|----------|
| HAP | 安装包基本单元 | 应用功能模块 |
| HAR | 静态共享包，编译时引用 | 公共库、工具类 |
| HSP | 动态共享包，运行时共享 | 多 HAP 共用的代码/资源 |

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

> 详细参考见 `references/electron-harmonyos.md`，此处为核心要点速查。

Electron for HarmonyOS 允许 Web 技术栈开发者复用现有代码构建鸿蒙 PC 应用，基于 Electron 34 + HarmonyOS Compatible SDK 5.0.5（API 17+）。

### 核心架构差异

| 对比维度 | 传统 Electron | 鸿蒙 Electron |
|----------|-------------|----------------|
| 渲染引擎 | Chromium（完整 Blink + V8） | 鸿蒙 Web 组件（基于 Chromium 深度定制） |
| JS 运行时 | Node.js（主进程）+ V8（渲染进程） | ArkCompiler + QuickJS（**无完整 Node.js**） |
| 进程模型 | 主进程 + 多渲染进程（沙箱隔离） | 单进程（Stage 模型）+ 鸿蒙 Ability |
| 原生能力访问 | Node.js API + Native Modules | 鸿蒙系统 API（@ohos.*）+ IPC 代理 |
| 文件系统 | 传统路径（C:/、/Users/） | 鸿蒙沙箱路径 |
| 打包格式 | .exe/.dmg/.AppImage | .hap（鸿蒙应用包） |

**关键结论：鸿蒙 Electron 不支持 Node.js 运行时**，所有原生能力需通过鸿蒙系统 API 实现，必须适配鸿蒙沙箱和权限模型。

### 必需配置

```javascript
// 1. 禁用硬件加速（避免黑屏/卡顿）
app.disableHardwareAcceleration()
app.commandLine.appendSwitch('disable-gpu')

// 2. 强制安全配置（不可关闭）
// webPreferences 中必须：
contextIsolation: true   // 开启上下文隔离
nodeIntegration: false   // 禁用 Node 集成
```

### 项目结构

```
ohos_hap/
├── web_engine/src/main/resources/resfile/resources/app/
│   ├── main.js            # Electron 主进程入口
│   ├── preload.js         # 预加载脚本（安全桥梁）
│   ├── index.html         # 渲染进程页面
│   ├── package.json       # 项目依赖配置
│   └── harmony-adapter.js # 鸿蒙适配层（Mock 不支持的模块）
├── libs/                  # 核心依赖库（含 libelectron.so）
└── module.json5           # 鸿蒙应用配置（权限、包名）
```

### 生命周期适配

| 传统 Electron | 鸿蒙 Stage 模型 | 适配逻辑 |
|---------------|----------------|----------|
| `app.ready()` | `onCreate()` | 先执行 `onCreate()` 初始化，再触发 `app.ready()` |
| `window.close()` | `onDestroy()` | 需调用 `terminateAbility()` 释放资源 |
| `app.quit()` | `onRelease()` | 退出前在 `onRelease()` 中保存数据 |

### 常见问题排查

| 问题 | 解决方案 |
|------|----------|
| 窗口黑屏 | 添加 `app.disableHardwareAcceleration()` + `disable-gpu` 开关 |
| 找不到 libelectron.so | 重新下载预编译包，确认 `libs/arm64-v8a` 目录存在 |
| CSP 阻止脚本执行 | `index.html` 的 `<meta>` 中配置正确的 CSP 策略 |
| 签名验证失败 | `Project Structure → Signing Configs → Reset` 重新生成 |
| Native 模块不可用 | 创建 `harmony-adapter.js` Mock 不支持的模块，用鸿蒙 API 替代 |

## 应用签名与发布

### 签名流程

1. 登录 AppGallery Connect 创建应用
2. 生成调试证书和调试 Profile（开发阶段）
3. 发布证书和发布 Profile（上架阶段）
4. 在 `build-profile.json5` 中配置签名信息

### 构建与上架

1. Build → Build Hap(s)/APP(s) → Build APP(s)
2. 生成 `.app` 格式发布包
3. 登录 AppGallery Connect 提交上架审核

## 开发工作流决策树

根据用户需求选择对应开发路径：

1. **从零创建鸿蒙应用** → 参照「开发环境搭建」+「工程结构」，生成完整脚手架
2. **编写 UI 页面** → 使用 ArkUI 声明式语法，参照「ArkUI 声明式 UI」+「装饰器速查」
3. **状态管理问题** → 查阅「装饰器速查」，区分 @State/@Prop/@Link/@Provide/@Consume 使用场景
4. **页面跳转/导航** → 简单场景用 Router，复杂场景用 Navigation，参照「页面路由与导航」
5. **网络请求** → 参照「网络请求」+「权限声明」
6. **数据存储** → 轻量用 Preferences，结构化数据用 RelationalStore
7. **调用系统能力** → 查阅「常用系统能力」模块表，必要时 WebFetch 官方 API 文档
8. **使用 Electron 开发鸿蒙 PC 应用** → 参照「Electron for HarmonyOS」+ `references/electron-harmonyos.md`
9. **迁移现有 Electron 项目到鸿蒙** → 参照 `references/electron-harmonyos.md` 中「项目迁移指南」
10. **TS/JS 代码翻译为 ArkTS** → 参照 `references/arkts-restrictions.md`（编译器错误速查表）
11. **应用签名/上架** → 参照「应用签名与发布」

## 参考文档

详细参考材料见 `references/` 目录：

- `references/arkts-guide.md`：ArkTS 语言详细语法参考
- `references/arkts-restrictions.md`：**ArkTS 编译器严格限制与错误对照表**（TS→ArkTS 翻译必读）
- `references/arkui-components.md`：ArkUI 组件属性和事件完整参考
- `references/project-templates.md`：常用工程模板和代码片段
- `references/electron-harmonyos.md`：Electron for HarmonyOS 完整开发指南（环境搭建、项目迁移、API 适配、性能优化）

当需要 API 细节或组件属性时，优先 WebFetch 官方文档获取最新内容，references 中的内容作为辅助。

## 弹窗最佳实践

### 三种弹窗选用指南

| 场景 | 推荐组件 | 原因 |
|------|---------|------|
| 简单确认/提示（是/否） | `AlertDialog` via `getUIContext().showAlertDialog()` | 系统样式，API 简单 |
| 列表选择（多选项） | `ActionSheet` via `getUIContext().showActionSheet()` | 原生底部列表，`sheets` 数组即可 |
| **需要自定义样式** | `@CustomDialog` + `CustomDialogController` | 完全控制布局和样式 |
| 轻量提示 | `promptAction.showToast()` | 非模态，自动消失 |

### ⚠️ ActionSheet 样式陷阱

`showActionSheet` 默认 `backgroundBlurStyle: BlurStyle.COMPONENT_ULTRA_THICK`，会与 `backgroundColor` 叠加产生异常效果。如需自定义背景色，必须同时设置：
```typescript
backgroundBlurStyle: BlurStyle.NONE,
backgroundColor: '#faf8f2',
```

**结论：需要真正自定义样式的弹窗，强烈推荐使用 `@CustomDialog`。**

### 📘 CustomDialog 标准模式

```typescript
// 1. 定义自定义弹窗
@CustomDialog
struct MyActionDialog {
  // ⚠️ controller 必须用可选声明 `?`（新版 ArkTS 编译器强制要求）
  // 只声明 `controller: CustomDialogController;`（无默认值）会触发编译错误：
  //   "If a component attribute supports local initialization,
  //    a valid, runtime-independent default value should be set for it"
  // 去掉 controller 属性也会报错："@CustomDialog must contain CustomDialogController property"
  // 正确方案：可选语法 `controller?` + `onClose` 回调关闭弹窗
  controller?: CustomDialogController;
  title: string = '';
  onConfirm: () => void = () => {};
  onCancel: () => void = () => {};
  onClose: () => void = () => {};   // ⚠️ 用回调关闭，避免依赖框架注入的 controller

  build() {
    Column() {
      Text(this.title).fontSize(16).padding(16)
      Row() {
        Button('取消').onClick(() => { this.onClose(); this.onCancel(); })
        Button('确定').onClick(() => { this.onClose(); this.onConfirm(); })
      }
    }
    .backgroundColor(Color.White).borderRadius(14)
    .padding(16)
  }
}

// 2. 在页面中打开
showMyDialog(data: MyData): void {
  // ⚠️ 必须显式类型注解！const ctrl = new ... 会推断为 any → 报 arkts-no-any-unknown
  const ctrl: CustomDialogController = new CustomDialogController({
    builder: MyActionDialog({
      title: data.name,
      onConfirm: () => { /* 确认逻辑 */ },
      onCancel: () => { /* 取消逻辑 */ },
      // ⚠️ 不能传 controller: ctrl（TDZ 错误 "used before its declaration"）
      // ⚠️ 箭头函数必须用 {} 函数体，不能用表达式 () => expr（arkts-no-implicit-return-types）
      onClose: () => { ctrl.close(); },
    }),
    alignment: DialogAlignment.Bottom,
    customStyle: true,       // 必须！关闭系统默认样式
    autoCancel: true,
    maskColor: 'rgba(0,0,0,0.4)',
  });
  ctrl.open();
}
```

### 关键要点
- `@CustomDialog` 装饰的结构体**必须**有 `CustomDialogController` 类型成员
- **⚠️ controller 声明必须用可选语法 `controller?: CustomDialogController`**（新版编译器不再接受无默认值声明）
- **⚠️ 关闭弹窗用 `onClose` 回调**，不要在 builder 中传 `controller: ctrl`（TDZ 报错），也不要依赖 `this.controller?.close()`
- **⚠️ `const ctrl = new CustomDialogController(...)` 必须加显式类型注解 `ctrl: CustomDialogController`**，否则推断为 any 触发编译错误
- **⚠️ 箭头函数在 builder 参数中需用函数体 `{}` 语法**，如 `onClose: () => { ctrl.close(); }` 而非 `onClose: () => ctrl.close()`
- 父组件创建 `CustomDialogController` 并传 builder 参数，调用 `.open()` 打开弹窗
- `customStyle: true` 关闭系统默认圆角/背景，完全由 build() 控制样式
- `alignment: DialogAlignment.Bottom` 实现底部弹出效果

### 常见编译错误速查

| 错误信息 | 原因 | 解决 |
|---------|------|------|
| `runtime-independent default value should be set for it` | `controller: CustomDialogController;` 无默认值 | 改为 `controller?: CustomDialogController;` |
| `must contain a property of the CustomDialogController type` | 缺少 controller 属性 | 必须保留 `controller?: CustomDialogController;` |
| `used before its declaration` | builder 中 `controller: ctrl` 引用了正在声明的变量 | 改用 `onClose: () => { ctrl.close(); }` 回调 |
| `Use explicit types instead of "any"` | `const ctrl = new ...` 无类型注解 | 加 `const ctrl: CustomDialogController = ...` |
| `Function return type inference is limited` | 箭头函数用表达式 `() => expr` | 改为函数体 `() => { ... }` |

### 官方参考
- 对话框概述：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-use-dialogs
- 自定义弹窗：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-common-components-custom-dialog
- 样式修复指南：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-fixes-style-dialog
- ActionSheet API：https://developer.huawei.com/consumer/cn/doc/harmonyos-references/ts-methods-action-sheet

## 版本历史

| 版本 | 日期 | 变更内容 |
|------|------|---------|
| 1.0.0 | — | 初始版本 |
| 1.0.1 | — | 增加 CustomComponent 属性名冲突规则（`scale` → `drawScale`）|
| 1.1.0 | — | 增加 AlertDialog/ActionSheet 废弃 API → UIContext 方法迁移规则 |
| 1.2.0 | — | 增加 Pillar 数据模型、Canvas 统一缩放、逆缩放字体等 |
| 1.3.0 | 2026-06-12 | **正确弹窗模式**：ActionSheet 样式陷阱文档化 + CustomDialog 标准模式。过时的 `AlertDialog.show()` 示例更新为 `getUIContext().showAlertDialog()`。补充弹窗选用指南和官方文档入口。 |
| 1.3.1 | 2026-06-12 | **⚠️ 修正 CustomDialog controller 自引用 bug**：`controller: CustomDialogController = new CustomDialogController({ builder: 自身 })` 导致 undefined。正确做法是只声明类型 `controller: CustomDialogController`（无默认值），框架自动注入实例。 |
| 1.4.0 | 2026-06-12 | **controller 改为可选声明**（`controller?: CustomDialogController` + `?.close()`），解决新版 ArkTS 编译器 "If a component attribute supports local initialization, a valid, runtime-independent default value should be set for it" 编译错误。 |
| 1.5.0 | 2026-06-12 | **CustomDialog 关闭改为 onClose 回调模式**：解决 `controller: ctrl` TDZ 错误、`const ctrl` 需显式类型注解避免 any 推断、箭头函数需用 `{}` 函数体语法。新增编译错误速查表。 |
| 1.5.1 | 2026-06-15 | **JSON 反序列化 `unknown` 修复**：`unknown` 参数类型改用 `Object`，`JSON.parse()` 返回值显式 `as Object`，数组变量显式 `as Object[]`，更新 arkts-restrictions.md 第 4 条。 |
