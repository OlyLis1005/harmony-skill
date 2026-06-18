# Electron for HarmonyOS 完整开发指南

> 来源：https://gitcode.com/openharmony-sig/electron + https://www.electronjs.org/docs/latest
> 优先以华为官方文档和 GitCode 仓库为准，本文档作为快速参考。

## 一、概述

Electron for HarmonyOS 是 openharmony-sig 社区基于 Electron 34 稳定版适配鸿蒙系统的框架，允许 Web 技术栈（HTML/CSS/JavaScript）开发者：

- **新项目**：使用 Electron API 直接开发鸿蒙 PC 应用
- **迁移项目**：将现有 Electron 应用低成本迁移到鸿蒙平台

当前版本基于：Electron 34 + HarmonyOS Compatible SDK 5.0.5（API 17+），目标设备为鸿蒙 NEXT PC/平板。

## 二、环境搭建

### 2.1 环境要求

| 工具/环境 | 版本要求 | 备注 |
|-----------|---------|------|
| 操作系统（开发） | Windows 10/11、macOS 10.15+、Ubuntu 22.04+ | Windows/macOS 用于开发运行，Ubuntu 用于编译 |
| DevEco Studio | 5.0.0.100+ | https://developer.huawei.com/consumer/cn/deveco-studio/ |
| Node.js | v18.x LTS 或 v20.x LTS | 推荐 nvm 管理版本 |
| 鸿蒙 SDK | Compatible SDK 5.0.5（API 17+） | DevEco Studio 内自动下载 |
| Electron 预编译包 | v34.6.0+（鸿蒙适配版） | 见下方获取方式 |
| 目标设备 | 鸿蒙 NEXT（API 20+）PC/平板 | 可使用鸿蒙模拟器 |

### 2.2 获取 Electron 鸿蒙版

**方式一：使用预编译包（推荐）**

适合不需要 Native Addon 和 ArkTS 调用的场景：

1. 登录华为 DevCloud：https://devcloud.cn-north-4.huaweicloud.com/codehub/project/b19f5ea8ffd4492ea8c06ca2ebf3f858/codehub/2821214/home?ref=electron34-release
2. 下载 Electron 34 最新日期的二进制 release 包
3. 解压即可使用

**方式二：从源码编译**

> 仅在需要自定义编译或使用 Native Addon 时使用。编译环境**必须 Ubuntu 22.04**，8 核虚拟机约需 8 小时。

```bash
# 1. 安装工具
sudo apt install -y git-lfs ccache curl python3 python-is-python3 python3-pip

# 2. 下载 repo 工具
mkdir ~/bin && curl https://gitee.com/oschina/repo/raw/fork_flow/repo-py3 > ~/bin/repo
chmod a+x ~/bin/repo && export PATH=~/bin:$PATH

# 3. 安装 Node.js（通过 NodeSource）
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# 4. 克隆代码
git clone -b master https://gitcode.com/openharmony-sig/electron.git
cd electron
git lfs pull

# 5. 拉取 Chromium 代码
repo init -u https://gitee.com/openharmony-sig/manifest.git -b pc_chromium_132 --partial-clone
repo sync -c -j8

# 6. 应用 Patch
./override_files.sh

# 7. 安装编译依赖
sudo ./src/build/install-build-deps.sh --no-chromeos-fonts

# 8. 执行编译
./electron_build.sh

# 9. 产物在 src/out/musl_64 目录
#    libelectron.so、libffmpeg.so、libadapter.so、electron 等
```

### 2.3 开发环境配置

1. 安装 DevEco Studio 5.0+
2. 配置 SDK：选择 Compatible SDK 5.0.5
3. **配置项目签名（必需）**：`File → Project Structure → Signing Configs → Auto-generate signature`
4. 配置环境变量：将 `{DevEco安装路径}\sdk\default\openharmony\toolchains` 添加到 PATH，确保 `hdc` 命令可用

### 2.4 导入预编译项目

1. 解压预编译包，得到 `ohos_hap` 目录
2. DevEco Studio → `File → Open` → 选择 `ohos_hap` 目录
3. 首次运行需登录华为账号生成证书
4. 运行后按 `Ctrl + Alt + I` 打开调试工具

## 三、核心架构与差异

### 3.1 传统 Electron vs 鸿蒙 Electron 对比

| 对比维度 | 传统 Electron | 鸿蒙 Electron |
|----------|-------------|----------------|
| 渲染引擎 | Chromium（完整 Blink + V8） | 鸿蒙 Web 组件（基于 Chromium 深度定制，移除冗余模块） |
| JS 运行时 | Node.js（主进程）+ V8（渲染进程） | ArkCompiler + QuickJS（兼容 V8 语法，**无完整 Node.js**） |
| 进程模型 | 主进程 + 多渲染进程（沙箱隔离） | 单进程（Stage 模型）+ 鸿蒙 Ability（轻量级隔离） |
| 原生能力访问 | Node.js API + Native Modules | 鸿蒙系统 API（@ohos.*）+ IPC 代理 |
| 文件系统 | 传统文件路径（C:/、/Users/） | 鸿蒙沙箱路径（应用仅访问自身沙箱目录） |
| 打包格式 | .exe/.dmg/.AppImage | .hap（鸿蒙应用包，支持多终端部署） |
| 安全机制 | preload + 上下文隔离（可选） | 鸿蒙权限系统 + 进程隔离 + **强制安全配置** |

### 3.2 关键约束

- **不支持 Node.js 运行时**：所有原生能力需通过鸿蒙系统 API 实现
- **强制安全配置**：`contextIsolation: true` 和 `nodeIntegration: false` 不可关闭
- **必须禁用硬件加速**：否则会出现黑屏/卡顿
- **文件路径受限**：只能访问鸿蒙沙箱目录
- **Native Addon 不可直接使用**：需用鸿蒙 API Mock 替代

## 四、第一个应用

### 4.1 项目目录结构

```
ohos_hap/
├── web_engine/
│   └── src/main/resources/resfile/resources/app/  # 开发者代码目录
│       ├── main.js            # Electron 主进程入口
│       ├── preload.js         # 预加载脚本
│       ├── index.html         # 渲染进程页面
│       ├── renderer.js        # 渲染进程逻辑
│       ├── package.json       # 项目依赖
│       └── harmony-adapter.js # 鸿蒙适配层（Mock 不支持的模块）
├── libs/                       # 核心依赖库（含 libelectron.so）
└── module.json5                # 鸿蒙应用配置（权限、包名）
```

### 4.2 main.js（主进程入口）

```javascript
const { app, BrowserWindow, ipcMain } = require('electron')
const path = require('path')

// ⚠️ 鸿蒙 PC 必需：禁用硬件加速
app.disableHardwareAcceleration()
app.commandLine.appendSwitch('disable-gpu')

let mainWindow = null

function createWindow() {
  mainWindow = new BrowserWindow({
    width: 900,
    height: 600,
    title: '鸿蒙Electron示例',
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true,    // 安全必需，不可关闭
      nodeIntegration: false,   // 安全必需，不可关闭
      webviewTag: true,
      devTools: true
    }
  })

  mainWindow.loadFile(path.join(__dirname, 'index.html'))
  mainWindow.webContents.openDevTools({ mode: 'right' })

  mainWindow.on('closed', () => {
    mainWindow = null
  })
}

app.whenReady().then(() => {
  createWindow()
  app.on('activate', () => {
    if (BrowserWindow.getAllWindows().length === 0) {
      createWindow()
    }
  })
})

app.on('window-all-closed', () => {
  app.quit()
})

// IPC 通信示例
ipcMain.handle('get-app-version', () => {
  return app.getVersion()
})

ipcMain.on('show-message', (event, msg) => {
  console.log('渲染进程消息：', msg)
  event.reply('message-reply', `主进程已收到：${msg}`)
})
```

### 4.3 preload.js（预加载脚本）

```javascript
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('ohosElectronAPI', {
  getAppVersion: () => ipcRenderer.invoke('get-app-version'),
  sendMessage: (msg) => ipcRenderer.send('show-message', msg),
  onMessageReply: (callback) => {
    const listener = (event, data) => callback(data)
    ipcRenderer.on('message-reply', listener)
    // 返回取消监听方法（避免内存泄漏）
    return () => ipcRenderer.removeListener('message-reply', listener)
  }
})
```

### 4.4 index.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- ⚠️ 鸿蒙必需：放宽 CSP 策略 -->
  <meta http-equiv="Content-Security-Policy"
    content="default-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data:;">
  <title>鸿蒙 Electron Hello World</title>
</head>
<body>
  <h1>Hello Electron on HarmonyOS!</h1>
  <div>应用版本：<span id="app-version">加载中...</span></div>
  <button onclick="testIPC()">测试IPC通信</button>
  <div id="message-reply" style="display:none;"></div>
  <script src="renderer.js"></script>
</body>
</html>
```

### 4.5 renderer.js

```javascript
document.addEventListener('DOMContentLoaded', async () => {
  const appVersion = await window.ohosElectronAPI.getAppVersion()
  document.getElementById('app-version').textContent = appVersion

  const removeListener = window.ohosElectronAPI.onMessageReply((data) => {
    const replyEl = document.getElementById('message-reply')
    replyEl.textContent = data
    replyEl.style.display = 'block'
  })

  window.addEventListener('beforeunload', removeListener)
})

function testIPC() {
  const msg = `测试消息（${new Date().toLocaleTimeString()}）`
  window.ohosElectronAPI.sendMessage(msg)
}
```

## 五、生命周期适配

### 5.1 生命周期对照

| 传统 Electron | 鸿蒙 Stage 模型 | 适配逻辑 |
|---------------|----------------|----------|
| `app.ready()` | `onCreate()` | 先执行 `onCreate()` 初始化，再触发 `app.ready()` 创建窗口 |
| `window.close()` | `onDestroy()` | 窗口关闭时需调用 `terminateAbility()` 释放资源 |
| `app.quit()` | `onRelease()` | 退出前在 `onRelease()` 中保存数据 |

### 5.2 生命周期适配代码

```javascript
const { app, BrowserWindow } = require('electron')

let mainWindow = null

// 鸿蒙 Ability 生命周期回调
const ohosAbility = require('@ohos.ability.uiAbility')

ohosAbility.on('create', () => {
  console.log('鸿蒙 Ability onCreate')
  initAppConfig()
})

app.whenReady().then(() => {
  // 鸿蒙适配：延迟 500ms 创建窗口，避免崩溃
  setTimeout(() => {
    createWindow()
    ohosAbility.on('destroy', () => {
      console.log('鸿蒙 Ability destroy')
      saveAppConfig()
      app.quit()
    })
  }, 500)
})

function initAppConfig() {
  // 初始化应用配置
}

function saveAppConfig() {
  // 保存应用配置
}
```

## 六、安全机制适配

### 6.1 强制安全配置

鸿蒙 Electron 以下配置不可关闭，开发者必须遵循：

```javascript
new BrowserWindow({
  webPreferences: {
    contextIsolation: true,    // 必须开启
    nodeIntegration: false,   // 必须关闭
  }
})
```

### 6.2 权限声明（module.json5）

访问敏感能力需在 `module.json5` 中声明权限：

```json
{
  "module": {
    "name": "web_engine",
    "type": "feature",
    "deviceTypes": ["phone", "tablet", "pc"],
    "reqPermissions": [
      {
        "name": "ohos.permission.READ_USER_STORAGE",
        "reason": "需要读取用户配置文件",
        "usedScene": {
          "ability": ["EntryAbility"],
          "when": "always"
        }
      },
      {
        "name": "ohos.permission.WRITE_USER_STORAGE",
        "reason": "需要保存用户配置文件",
        "usedScene": {
          "ability": ["EntryAbility"],
          "when": "always"
        }
      }
    ]
  }
}
```

### 6.3 preload 安全模式

```javascript
const { contextBridge, ipcRenderer } = require('electron')

// 白名单 IPC 通道
const validInvokeChannels = ['db:open', 'db:save', 'config:get', 'config:set']
const validSendChannels = ['window:minimize', 'window:maximize', 'window:close']
const validOnChannels = ['db:changed', 'notification:show']

contextBridge.exposeInMainWorld('myAPI', {
  invoke: (channel, ...args) => {
    if (!validInvokeChannels.includes(channel)) {
      throw new Error(`非法IPC通道：${channel}`)
    }
    return ipcRenderer.invoke(channel, ...args)
  },

  send: (channel, ...args) => {
    if (!validSendChannels.includes(channel)) return
    ipcRenderer.send(channel, ...args)
  },

  on: (channel, callback) => {
    if (!validOnChannels.includes(channel)) {
      throw new Error(`非法IPC通道：${channel}`)
    }
    const listener = (event, ...args) => callback(...args)
    ipcRenderer.on(channel, listener)
    return () => ipcRenderer.removeListener(channel, listener)
  }
})
```

## 七、文件路径适配

### 7.1 路径映射

| `app.getPath()` 参数 | 传统 Electron 路径 | 鸿蒙 Electron 路径 | 用途 |
|---|---|---|---|
| `userData` | C:/Users/XXX/AppData/Roaming/XXX | /data/storage/el2/base/haps/XXX/data/userdata/ | 用户配置、缓存 |
| `temp` | C:/Users/XXX/AppData/Local/Temp/XXX | /data/storage/el2/base/haps/XXX/data/temp/ | 临时文件 |
| `documents` | C:/Users/XXX/Documents | /data/storage/el2/base/haps/XXX/data/documents/ | 用户文档 |
| `downloads` | C:/Users/XXX/Downloads | /data/storage/el2/base/haps/XXX/data/downloads/ | 下载文件 |

### 7.2 文件操作适配代码

```javascript
const { app } = require('electron')
const path = require('path')
const fs = require('fs').promises

const userDataPath = app.getPath('userData')
const configPath = path.join(userDataPath, 'app_config.json')

async function writeConfig(config) {
  try {
    await fs.mkdir(path.dirname(configPath), { recursive: true })
    await fs.writeFile(configPath, JSON.stringify(config, null, 2), 'utf8')
  } catch (err) {
    console.error('配置写入失败：', err)
  }
}

async function readConfig() {
  try {
    const data = await fs.readFile(configPath, 'utf8')
    return JSON.parse(data)
  } catch (err) {
    if (err.code === 'ENOENT') {
      return { theme: 'light', language: 'zh-CN' } // 默认配置
    }
    console.error('配置读取失败：', err)
    return null
  }
}
```

## 八、Native 模块适配（Mock 模式）

鸿蒙 Electron 不支持 Node.js Native Addon，需创建适配层 Mock 替代。

### 8.1 harmony-adapter.js 示例（Mock keytar）

```javascript
// harmony-adapter.js
const ohosSecureStorage = require('@ohos/security.storage')

const mockKeytar = {
  getPassword: async (service, account) => {
    try {
      const key = `${service}_${account}`
      const password = await ohosSecureStorage.get(key)
      return password || null
    } catch (err) {
      console.error('keytar getPassword 失败：', err)
      return null
    }
  },

  setPassword: async (service, account, password) => {
    try {
      const key = `${service}_${account}`
      await ohosSecureStorage.set(key, password)
    } catch (err) {
      console.error('keytar setPassword 失败：', err)
      throw err
    }
  },

  deletePassword: async (service, account) => {
    try {
      const key = `${service}_${account}`
      await ohosSecureStorage.delete(key)
      return true
    } catch (err) {
      console.error('keytar deletePassword 失败：', err)
      return false
    }
  }
}

// 替换 require 缓存
require.cache.keytar = { exports: mockKeytar }
global.keytar = mockKeytar
```

### 8.2 在主进程入口加载适配层

```javascript
// main.js（顶部添加，必须在 require 原模块之前）
require('../harmony-adapter.js')
const keytar = require('keytar')  // 此时加载的是 mockKeytar
```

## 九、项目迁移指南

### 9.1 迁移方案选择

| 方案 | 原理 | 优势 | 适用场景 |
|------|------|------|----------|
| **方案 A：鸿蒙 Web 组件迁移** | 用鸿蒙原生 Web 组件替代 BrowserWindow | 性能优、鸿蒙生态适配好 | 新项目、简单 Electron 应用 |
| **方案 B：直接复用迁移** | 通过鸿蒙 Electron 适配器直接运行现有代码 | 零成本复用、开发效率高 | 复杂 Electron 应用（含 Native 模块） |

### 9.2 迁移步骤（方案 B）

1. **项目结构调整**：将原始代码放入 `web_engine/src/main/resources/resfile/resources/app/`
2. **package.json 适配**：替换不兼容依赖，添加鸿蒙特有包
3. **主进程适配**：
   - 添加 `app.disableHardwareAcceleration()` + `disable-gpu`
   - 适配生命周期（延迟创建窗口、对齐 Ability 回调）
   - Mock 不支持的 Native 模块
4. **预加载脚本适配**：使用 `contextBridge` 安全暴露 IPC API
5. **CSP 策略调整**：在 HTML 中配置正确的 Content-Security-Policy
6. **文件路径适配**：使用 `app.getPath()` 获取鸿蒙沙箱路径
7. **权限声明**：在 `module.json5` 中声明所需权限

### 9.3 package.json 适配示例

```json
{
  "name": "myapp-ohos",
  "version": "1.0.0",
  "main": "./main/main.js",
  "scripts": {
    "start": "electron .",
    "build:ohos": "ohos-builder build --output ./dist"
  },
  "dependencies": {
    "electron": "^34.6.0"
  },
  "devDependencies": {
    "@ohos/hap-builder": "^5.0.0"
  },
  "ohos": {
    "deviceTypes": ["pc", "tablet"],
    "hapName": "MyApp-ohos"
  }
}
```

## 十、Electron 核心 API 速查

### 10.1 主进程模块（鸿蒙 Electron 已支持的常用 API）

| 模块 | 用途 | 鸿蒙适配状态 |
|------|------|-------------|
| `app` | 应用生命周期、事件 | ✅ 已适配 |
| `BrowserWindow` | 创建和管理应用窗口 | ✅ 已适配（需禁用硬件加速） |
| `ipcMain` | 主进程 IPC 通信 | ✅ 已适配 |
| `Menu` / `MenuItem` | 原生菜单 | ⚠️ 部分适配 |
| `dialog` | 原生对话框 | ⚠️ 部分适配 |
| `Tray` | 系统托盘 | ✅ 已适配 |
| `nativeImage` | 原生图片 | ✅ 已适配 |
| `session` | 会话管理 | ⚠️ 部分适配 |
| `shell` | 系统壳层操作 | ⚠️ 部分适配 |
| `screen` | 屏幕信息 | ⚠️ 部分适配 |
| `clipboard` | 剪贴板 | ⚠️ 部分适配 |
| `net` | 网络请求 | ✅ 已适配 |
| `protocol` | 协议注册 | ⚠️ 部分适配 |
| `globalShortcut` | 全局快捷键 | ⚠️ 部分适配 |
| `autoUpdater` | 自动更新 | ❌ 未适配 |
| `Notification` | 系统通知 | ⚠️ 部分适配 |
| `powerMonitor` | 电源监控 | ❌ 未适配 |
| `webContents` | 网页内容管理 | ✅ 已适配 |

> ⚠️ 适配状态基于社区信息整理，具体以 https://gitcode.com/openharmony-sig/electron 最新版本为准。

### 10.2 渲染进程模块

| 模块 | 用途 | 鸿蒙适配状态 |
|------|------|-------------|
| `contextBridge` | 安全暴露 API 到渲染进程 | ✅ 已适配 |
| `ipcRenderer` | 渲染进程 IPC 通信 | ✅ 已适配 |
| `webFrame` | 网页框架控制 | ⚠️ 部分适配 |

### 10.3 IPC 通信模式

```javascript
// === 主进程 ===
const { ipcMain } = require('electron')

// invoke/handle 模式（双向，推荐）
ipcMain.handle('channel-name', async (event, ...args) => {
  return result
})

// send/on 模式（单向或双向）
ipcMain.on('channel-name', (event, data) => {
  event.reply('reply-channel', responseData)
})

// === 渲染进程（通过 preload） ===
const { contextBridge, ipcRenderer } = require('electron')

contextBridge.exposeInMainWorld('api', {
  // invoke 模式
  invokeMethod: (...args) => ipcRenderer.invoke('channel-name', ...args),
  // send 模式
  sendData: (data) => ipcRenderer.send('channel-name', data),
  // 监听主进程消息
  onMessage: (callback) => {
    const listener = (event, ...args) => callback(...args)
    ipcRenderer.on('reply-channel', listener)
    return () => ipcRenderer.removeListener('reply-channel', listener)
  }
})
```

## 十一、性能优化

### 11.1 渲染性能

```javascript
// 发布版本关闭 DevTools
webPreferences: { devTools: false }

// 限制渲染进程内存
app.commandLine.appendSwitch('js-flags', '--max-old-space-size=512')
```

### 11.2 内存优化

- 及时取消 IPC 监听，避免内存泄漏
- 窗口关闭时将引用设为 `null`
- 使用 `webContents.session.clearCache()` 定期清理缓存

### 11.3 启动速度

- 延迟加载非关键模块
- 压缩静态资源（terser 压缩 JS，csso 压缩 CSS）
- 使用 `require` 动态加载按需引入

## 十二、调试技巧

| 调试方式 | 方法 |
|----------|------|
| 渲染进程调试 | `mainWindow.webContents.openDevTools()` 或 `Ctrl + Alt + I` |
| 主进程日志 | DevEco Studio → Log 面板（过滤 `Electron` 标签） |
| 鸿蒙系统日志 | `hdc logcat` 命令查看 |
| VS Code 调试 | 配置 launch.json attach 到主进程 |

## 十三、常见问题排查

| 报错信息 | 解决方案 |
|----------|----------|
| 「SysCap 不匹配：缺少 SystemCapability」 | 编辑 `module.json5`，移除 `reqSysCapabilities` 中的测试类能力 |
| 「找不到 libelectron.so」 | 重新下载预编译包，确认 `libs/arm64-v8a` 目录包含 `libelectron.so` |
| 窗口黑屏/空白 | 添加 `app.disableHardwareAcceleration()` + `app.commandLine.appendSwitch('disable-gpu')` |
| 「CSP 阻止脚本执行」 | `index.html` 的 `<meta>` 中配置正确的 CSP 策略 |
| 「签名验证失败」 | `Project Structure → Signing Configs → Reset` 重新生成 |
| Native 模块加载失败 | 创建 `harmony-adapter.js` Mock 替代，用鸿蒙 API 实现等效功能 |
| 模拟器不支持 WebGL | 使用真机调试，或避免使用 WebGL 相关功能 |

## 十四、学习资源

| 资源 | 链接 |
|------|------|
| 鸿蒙 Electron 源码仓库 | https://gitcode.com/openharmony-sig/electron |
| Electron 官方文档 | https://www.electronjs.org/docs/latest |
| HarmonyOS 官方文档 | https://developer.huawei.com/consumer/cn/doc/ |
| HarmonyOS Electron 开发指南 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/electron-integration-0000001821453245 |
| Electron 社区讨论 | https://developer.huawei.com/consumer/cn/forum/block/electron |
| 鸿蒙 PC 开发者论坛 | https://harmonyospc.csdn.net/ |
| Electron 示例项目 | https://github.com/electron-for-harmonyos/samples |
| hdc 工具文档 | https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hdc-usage-0000001821453251 |

---

## 十五、Electron ↔ ArkTS 互调用

> 来源：https://gitcode.com/openharmony-sig/electron + `docs/call-arkts-function-guide/README.md`

### 15.1 systemPreferences.callArkTSFunction()

在鸿蒙 Electron 中，`systemPreferences.callArkTSFunction()` 提供了从 **Electron JS → ArkTS** 的跨语言调用通道，底层通过 AKI 框架将 ArkTS 函数桥接到 C++，再由 C++ 层暴露给 Electron 主进程的 V8。

**JS 端签名**：
```javascript
systemPreferences.callArkTSFunction(
  functionName: string,
  returnType?: string,
  paramArray?: any[]
): Promise<{ type: string, value: any }>
```

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `functionName` | `string` | 是 | AKI 注册的函数名，格式 `"模块名.方法名"`，如 `"EtsBridge.TestReturnString"` |
| `returnType` | `string` | 否 | 期望返回值类型：`"void"` / `"number"` / `"boolean"` / `"string"` / `"string[]"` / `"number[]"`，默认 `"void"` |
| `paramArray` | `any[]` | 否 | 参数数组。数组类型参数需嵌套：`[[1,2,3]]` 表示传递一个数组参数 |

**返回值**：`Promise<{ type: string, value: any }>`

### 15.2 JS 端调用示例

```javascript
const { systemPreferences } = require('electron');

// 无参数，返回字符串
const res = await systemPreferences.callArkTSFunction(
  'EtsBridge.TestReturnString', 'string');
// res = { type: "string", value: "ets bridge adapter" }

// 带参数调用
const res2 = await systemPreferences.callArkTSFunction(
  'EtsBridge.TestTwoParams', 'string', ['hello', 42]);
// res2 = { type: "string", value: "hello_42" }

// 返回字符串数组（注意外层嵌套数组）
const res3 = await systemPreferences.callArkTSFunction(
  'EtsBridge.TestReturnArrayString', 'string[]');
// res3 = { type: "string[]", value: ["ets", "bridge", "adapter"] }
```

### 15.3 ArkTS 端注册新函数

**步骤 1**：在 Adapter 中实现业务方法（`EtsBridgeAdapter.ets`）

```typescript
import { injectable } from 'inversify';
import { BaseAdapter } from '../common/BaseAdapter';
import LogMethod from '../common/LogDecorator';

@injectable()
export class EtsBridgeAdapter extends BaseAdapter {
  @LogMethod
  myCustomFunction(param1: string, param2: number): string {
    return `Result: ${param1} - ${param2}`;
  }
}
```

**步骤 2**：在 Bind 文件中创建包装函数并通过 AKI 注册（`EtsBridgeAdapterBind.ets`）

```typescript
import JsBindingUtils from '../utils/JsBindingUtils';
import Inject from '../common/InjectModule';
import lazy { EtsBridgeAdapter } from '../adapter/EtsBridgeAdapter';

function implEtsBridgeAdapter(): EtsBridgeAdapter {
  return Inject.getOrCreate(EtsBridgeAdapter);
}

// 包装函数 —— 签名必须与 Adapter 方法完全一致
function myCustomFunction(param1: string, param2: number): string {
  return implEtsBridgeAdapter().myCustomFunction(param1, param2);
}

export class EtsBridgeAdapterBind {
  static bind() {
    // 注册到 AKI，C++ 通过 GetJSFunction("EtsBridge.MyCustomFunction") 检索
    JsBindingUtils.bindFunction("EtsBridge.MyCustomFunction", myCustomFunction);
  }
}
```

**步骤 3**：确保 `bind()` 在 ArkTS 启动阶段被调用（`JsBindingMethod.ets`）

```typescript
import { EtsBridgeAdapterBind } from '../jsbindings/EtsBridgeAdapterBind';

export class JsBindingMethod {
  static bindAll() {
    // ... 其他 Adapter 的 bind()
    EtsBridgeAdapterBind.bind();
  }
}
```

### 15.4 已知限制

| 限制 | 说明 |
|------|------|
| **不支持异步调用** | ArkTS 函数必须同步返回，不能是 `async` 或返回 `Promise<T>` |
| **不支持 Function 参数** | 无法把 JS 回调函数传给 ArkTS |
| **不支持结构化 Object 参数** | Object 会被 JSON.stringify 降级为字符串，ArkTS 侧需自行 `JSON.parse` |
| **超过 3 个参数退化** | 4+ 参数时所有参数被转为 `string`（`InvokeWithStringArgs` 回退路径） |
| **数组参数需嵌套** | `[[1,2,3]]` 表示 1 个 `Array<number>` 参数，`[1,2,3]` 会被解析为 3 个独立参数 |

---

## 十六、HNP 打包与子进程

> 来源：`docs/hnp-packaging-guide/README.md`

### 16.1 背景：为什么需要 HNP

PC 25 镜像及以后对应用可执行二进制在内核层进行权限管控，**没有签名的二进制会被 xpm 拦截**。HNP（Harmony Native Package）方案用于对应用内可执行二进制进行签名，避免被 xpm 拦截。

### 16.2 HNP 包构建

**目录结构**：
```
hnp
├── bin
│   └── electron
│       └── electron          # 关键可执行文件
│       └── locales/          # 资源文件
│       └── ...
└── hnp.json
```

**hnp.json**：
```json
{
    "type": "hnp-config",
    "name": "electron",
    "version": "1.0",
    "install": {
        "links": [
            {
                "source": "/bin/electron",
                "target": "electron"
            }
        ]
    }
}
```

**打包命令**（在 DevEco 的 `toolchains` 目录下执行）：
```bash
hnpcli pack -i 实际路径/hnp -o hnp包目标路径 -n electron -v 1.0
```

### 16.3 module.json5 配置

```json
{
  "module": {
    "hnpPackages": [
      {
        "package": "electron.hnp",
        "type": "private"
      }
    ]
  }
}
```

### 16.4 fork 子进程

```javascript
const { fork } = require('child_process');
const path = require('path');

const child = fork(path.join(__dirname, 'child.js'));

child.on('message', (message) => {
    console.log('主进程收到消息', message);
});

child.send({ hello: 'from main process' });
```

### 16.5 spawn 调用可执行二进制

> ⚠️ 建议使用沙箱物理路径：`/data/app/electron.org/electron_1.0/bin/electron/hello`

```javascript
const { spawn } = require('child_process');

const binPath = "/data/app/electron.org/electron_1.0/bin/electron/hello";
const child = spawn(binPath, args, {
    cwd: path.dirname(binPath),
    stdio: ["ignore", "pipe", "pipe"]
});

let output = "";
child.stdout.on("data", data => { output += data.toString(); });
child.on("close", code => {
    if (code === 0) console.log(output);
});
```
