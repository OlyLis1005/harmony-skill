# 常用工程模板与代码片段

> 来源：https://developer.huawei.com/consumer/cn/doc/
> 优先以官方文档为准，本文档作为快速参考。

## 1. 空白应用模板（Empty Ability）

最基础的模板，只有一个 EntryAbility 和一个空白页面。

### EntryAbility.ets

```typescript
import { UIAbility, AbilityConstant, Want } from '@kit.AbilityKit'
import { window } from '@kit.ArkUI'

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    console.info('[EntryAbility] onCreate')
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    console.info('[EntryAbility] onWindowStageCreate')
    // 加载主页面
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        console.error(`Failed to load content: ${JSON.stringify(err)}`)
        return
      }
      console.info('[EntryAbility] loadContent succeed')
    })
  }

  onForeground(): void {
    console.info('[EntryAbility] onForeground')
  }

  onBackground(): void {
    console.info('[EntryAbility] onBackground')
  }

  onDestroy(): void {
    console.info('[EntryAbility] onDestroy')
  }
}
```

### Index.ets

```typescript
@Entry
@Component
struct Index {
  build() {
    Column() {
      Text('Hello World')
        .fontSize(30)
        .fontWeight(FontWeight.Bold)
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

## 2. 列表+详情模板

常见的列表页跳转详情页模式。

### ListPage.ets

```typescript
import router from '@ohos.router'

interface Article {
  id: number
  title: string
  summary: string
}

@Entry
@Component
struct ListPage {
  @State articles: Article[] = [
    { id: 1, title: '文章一', summary: '这是文章一的摘要' },
    { id: 2, title: '文章二', summary: '这是文章二的摘要' },
  ]

  build() {
    Column() {
      // 顶部栏
      Row() {
        Text('文章列表')
          .fontSize(20)
          .fontWeight(FontWeight.Bold)
      }
      .width('100%')
      .height(56)
      .padding({ left: 16, right: 16 })
      .justifyContent(FlexAlign.Center)

      // 列表
      List({ space: 8 }) {
        ForEach(this.articles, (item: Article) => {
          ListItem() {
            Row() {
              Column() {
                Text(item.title)
                  .fontSize(16)
                  .fontWeight(FontWeight.Medium)
                Text(item.summary)
                  .fontSize(14)
                  .fontColor('#999')
                  .margin({ top: 4 })
                  .maxLines(1)
                  .textOverflow({ overflow: TextOverflow.Ellipsis })
              }
              .layoutWeight(1)
              .alignItems(HorizontalAlign.Start)

              Text('>')
                .fontSize(16)
                .fontColor('#ccc')
            }
            .width('100%')
            .padding(16)
            .backgroundColor(Color.White)
            .borderRadius(8)
          }
          .onClick(() => {
            router.pushUrl({
              url: 'pages/DetailPage',
              params: { id: item.id, title: item.title }
            })
          })
        }, (item: Article) => item.id.toString())
      }
      .layoutWeight(1)
      .padding({ left: 16, right: 16, top: 8 })
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#f5f5f5')
  }
}
```

### DetailPage.ets

```typescript
import router from '@ohos.router'

@Entry
@Component
struct DetailPage {
  @State title: string = ''
  @State id: number = 0

  aboutToAppear() {
    const params = router.getParams() as Record<string, Object>
    this.id = params.id as number
    this.title = params.title as string
  }

  build() {
    Column() {
      // 顶部栏
      Row() {
        Text('< 返回')
          .fontSize(16)
          .fontColor('#007DFF')
          .onClick(() => { router.back() })
        Text(this.title)
          .fontSize(18)
          .fontWeight(FontWeight.Bold)
          .layoutWeight(1)
          .textAlign(TextAlign.Center)
        Text('   ')  // 占位对齐
          .fontSize(16)
      }
      .width('100%')
      .height(56)
      .padding({ left: 16, right: 16 })
      .alignItems(VerticalAlign.Center)

      // 内容
      Scroll() {
        Column() {
          Text(`文章 ID: ${this.id}`)
            .fontSize(16)
            .margin({ bottom: 12 })
          Text('这里是文章详情内容...')
            .fontSize(14)
            .fontColor('#666')
            .lineHeight(24)
        }
        .padding(16)
      }
      .layoutWeight(1)
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#ffffff')
  }
}
```

## 3. 网络请求封装

```typescript
import http from '@ohos.net.http'

interface RequestOptions {
  url: string
  method?: http.RequestMethod
  header?: Record<string, string>
  params?: Record<string, string | number>
  body?: string
  connectTimeout?: number
  readTimeout?: number
}

interface ApiResponse<T> {
  code: number
  message: string
  data: T
}

class HttpClient {
  private baseUrl: string = 'https://api.example.com'

  async request<T>(options: RequestOptions): Promise<ApiResponse<T>> {
    const httpRequest = http.createHttp()

    // 拼接 URL 和参数
    let url = options.url.startsWith('http') ? options.url : this.baseUrl + options.url
    if (options.params) {
      const query = Object.entries(options.params)
        .map(([k, v]) => `${k}=${encodeURIComponent(v.toString())}`)
        .join('&')
      url += (url.includes('?') ? '&' : '?') + query
    }

    try {
      const response = await httpRequest.request(url, {
        method: options.method ?? http.RequestMethod.GET,
        header: {
          'Content-Type': 'application/json',
          ...options.header,
        },
        extraData: options.body,
        connectTimeout: options.connectTimeout ?? 60000,
        readTimeout: options.readTimeout ?? 60000,
      })

      if (response.responseCode === 200) {
        return JSON.parse(response.result as string) as ApiResponse<T>
      }
      throw new Error(`HTTP ${response.responseCode}`)
    } finally {
      httpRequest.destroy()
    }
  }

  get<T>(url: string, params?: Record<string, string | number>): Promise<ApiResponse<T>> {
    return this.request<T>({ url, method: http.RequestMethod.GET, params })
  }

  post<T>(url: string, body?: string): Promise<ApiResponse<T>> {
    return this.request<T>({ url, method: http.RequestMethod.POST, body })
  }
}

export const httpClient = new HttpClient()
```

## 4. Preferences 存储封装

```typescript
import preferences from '@ohos.data.preferences'
import { Context } from '@kit.AbilityKit'

class StorageService {
  private store: preferences.Preferences | null = null

  async init(context: Context, name: string = 'app_store'): Promise<void> {
    this.store = await preferences.getPreferences(context, name)
  }

  async put(key: string, value: preferences.ValueType): Promise<void> {
    if (!this.store) throw new Error('Storage not initialized')
    await this.store.put(key, value)
    await this.store.flush()
  }

  async get(key: string, defaultValue: preferences.ValueType): Promise<preferences.ValueType> {
    if (!this.store) throw new Error('Storage not initialized')
    return await this.store.get(key, defaultValue)
  }

  async remove(key: string): Promise<void> {
    if (!this.store) throw new Error('Storage not initialized')
    await this.store.delete(key)
    await this.store.flush()
  }

  async clear(): Promise<void> {
    if (!this.store) throw new Error('Storage not initialized')
    await this.store.clear()
    await this.store.flush()
  }
}

export const storageService = new StorageService()
```

## 5. 主题配色方案

```typescript
// 亮色主题
export const LightTheme = {
  primary: '#007DFF',
  primaryDark: '#0066CC',
  background: '#FFFFFF',
  surface: '#F5F5F5',
  textPrimary: '#1A1A1A',
  textSecondary: '#666666',
  textTertiary: '#999999',
  divider: '#EEEEEE',
  error: '#E84026',
  success: '#64BB5C',
  warning: '#FF9800',
}

// 暗色主题
export const DarkTheme = {
  primary: '#3B82F6',
  primaryDark: '#2563EB',
  background: '#1A1A1A',
  surface: '#2D2D2D',
  textPrimary: '#FFFFFF',
  textSecondary: '#CCCCCC',
  textTertiary: '#999999',
  divider: '#3D3D3D',
  error: '#FF6B6B',
  success: '#4ADE80',
  warning: '#FBBF24',
}
```

## 6. module.json5 配置参考

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": ["phone", "tablet", "2in1"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "description": "$string:EntryAbility_desc",
        "icon": "$media:layered_image",
        "label": "$string:EntryAbility_label",
        "startWindowIcon": "$media:startIcon",
        "startWindowBackground": "$color:start_window_background",
        "exported": true,
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]
          }
        ]
      }
    ],
    "requestPermissions": [
      { "name": "ohos.permission.INTERNET" }
    ]
  }
}
```

## 7. app.json5 配置参考

```json5
{
  "app": {
    "bundleName": "com.example.myapp",
    "vendor": "example",
    "versionCode": 1000000,
    "versionName": "1.0.0",
    "icon": "$media:app_icon",
    "label": "$string:app_name"
  }
}
```
