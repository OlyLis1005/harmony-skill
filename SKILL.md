---
name: harmony-skill
description: 鸿蒙（HarmonyOS NEXT）应用开发技能包。当用户需要开发鸿蒙应用、编写 ArkTS/ArkUI 代码、创建鸿蒙工程、调试鸿蒙应用、使用 Electron 框架开发鸿蒙 PC 应用、将现有 Electron 项目迁移到鸿蒙、或咨询鸿蒙开发相关问题时触发此技能。覆盖环境搭建、工程结构、ArkTS 语法、ArkUI 声明式 UI、Electron for HarmonyOS、应用模型（Stage 模型）、路由导航、网络请求、数据持久化、应用签名与发布等全流程开发场景。以华为官方文档为准。
agent_created: true
---

# 鸿蒙应用开发

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

ArkTS 基于 TypeScript 扩展，是 HarmonyOS NEXT 的主要开发语言。

### 基本语法要点

- 使用严格类型检查，禁止 any 类型
- 不支持 any/unknown，需显式类型注解
- 状态变量必须类型声明，禁止省略
- 使用 `@Observed` 和 `@ObjectLink` 管理嵌套对象

### 装饰器速查

| 装饰器 | 作用 | 数据流向 |
|--------|------|----------|
| `@Entry` | 标记页面入口组件 | — |
| `@Component` | 标记自定义组件 | — |
| `@State` | 组件内部可变状态 | 本组件内 |
| `@Prop` | 父→子单向数据传递 | 父→子 |
| `@Link` | 父子双向数据绑定 | 父↔子 |
| `@Provide` | 跨层级向下提供数据 | 祖先→后代 |
| `@Consume` | 跨层级向上消费数据 | 后代←祖先 |
| `@Observed` | 标记可观察的类 | — |
| `@ObjectLink` | 嵌套对象状态管理 | 双向 |
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

// 自定义对话框
AlertDialog.show({ title: '提示', message: '确认操作？' })
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
10. **应用签名/上架** → 参照「应用签名与发布」

## 参考文档

详细参考材料见 `references/` 目录：

- `references/arkts-guide.md`：ArkTS 语言详细语法参考
- `references/arkui-components.md`：ArkUI 组件属性和事件完整参考
- `references/project-templates.md`：常用工程模板和代码片段
- `references/electron-harmonyos.md`：Electron for HarmonyOS 完整开发指南（环境搭建、项目迁移、API 适配、性能优化）

当需要 API 细节或组件属性时，优先 WebFetch 官方文档获取最新内容，references 中的内容作为辅助。
