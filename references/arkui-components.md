# ArkUI 组件属性与事件参考

> 来源：https://developer.huawei.com/consumer/cn/doc/harmonyos-references/arkui-ts
> 优先以官方文档为准，本文档作为快速参考。

## 1. 通用属性

所有组件共享的通用属性：

| 类别 | 属性 | 说明 |
|------|------|------|
| 尺寸 | `.width()` / `.height()` | 宽高，支持数字/百分比/字符串 |
| 尺寸 | `.size({ width: , height: })` | 同时设置宽高 |
| 边距 | `.margin()` | 外边距 |
| 边距 | `.padding()` | 内边距 |
| 背景 | `.backgroundColor()` | 背景颜色 |
| 背景 | `.backgroundImage()` | 背景图片 |
| 边框 | `.border()` | 边框样式 |
| 边框 | `.borderRadius()` | 圆角 |
| 显示 | `.visibility()` | 可见性（Visible/Hidden/None） |
| 显示 | `.opacity()` | 透明度 0~1 |
| 启用 | `.enabled()` | 是否启用 |
| 变换 | `.rotate()` / `.scale()` / `.translate()` | 变换 |
| 动画 | `.animation()` | 属性动画 |
| 手势 | `.gesture()` | 手势绑定 |
| 响应 | `.responseRegion()` | 响应区域 |
| 层级 | `.zIndex()` | 层叠顺序 |
| 约束 | `.constraintSize()` | 约束尺寸（min/max） |
| 位置 | `.position()` / `.offset()` | 绝对定位/偏移 |
| 阴影 | `.shadow()` | 阴影 |
| 裁剪 | `.clip()` | 裁剪溢出内容 |

## 2. 布局组件详解

### Column（垂直布局）

```typescript
Column() {
  Text('Item 1')
  Text('Item 2')
}
.width('100%')
.height('100%')
.justifyContent(FlexAlign.Center)     // 主轴对齐
.alignItems(HorizontalAlign.Center)   // 交叉轴对齐
```

### Row（水平布局）

```typescript
Row() {
  Text('Left')
  Text('Right')
}
.width('100%')
.justifyContent(FlexAlign.SpaceBetween)
.alignItems(VerticalAlign.Center)
```

### Stack（层叠布局）

```typescript
Stack({ alignContent: Alignment.Center }) {
  Image($r('app.media.bg')).width(200).height(200)
  Text('Overlay').fontColor(Color.White)
}
```

### List（列表）

```typescript
List({ space: 10 }) {
  ForEach(this.dataList, (item: DataModel) => {
    ListItem() {
      Row() {
        Text(item.title)
      }
    }
    .onClick(() => { /* 跳转详情 */ })
  }, (item: DataModel) => item.id.toString())
}
.divider({ strokeWidth: 1, color: '#eee' })
.edgeEffect(EdgeEffect.Spring)
```

### Grid（网格布局）

```typescript
Grid() {
  ForEach(this.items, (item: Item) => {
    GridItem() {
      Text(item.name)
    }
  })
}
.columnsTemplate('1fr 1fr 1fr')  // 三列
.rowsGap(10)
.columnsGap(10)
```

### Tabs（标签页）

```typescript
Tabs({ barPosition: BarPosition.Start }) {
  TabContent() {
    Text('首页内容')
  }.tabBar('首页')

  TabContent() {
    Text('我的内容')
  }.tabBar('我的')
}
.vertical(false)
.scrollable(true)
.barMode(BarMode.Fixed)
```

## 3. 基础组件详解

### Text

```typescript
Text('Hello World')
  .fontSize(16)
  .fontColor('#333')
  .fontWeight(FontWeight.Bold)
  .fontStyle(FontStyle.Normal)
  .fontFamily('HarmonyOS Sans')
  .textAlign(TextAlign.Center)
  .maxLines(2)
  .textOverflow({ overflow: TextOverflow.Ellipsis })
  .decoration({ type: TextDecorationType.None })
  .lineHeight(24)
  .letterSpacing(1)
  .copyOption(CopyOptions.LocalDevice)  // 可复制
```

### Image

```typescript
Image($r('app.media.pic'))     // Resource 引用
  // 或 Image('https://xxx.png')  // 网络
  // 或 Image('file:///data/...')  // 本地
  .width(200)
  .height(200)
  .objectFit(ImageFit.Cover)      // 缩放模式
  .interpolation(ImageInterpolation.High)  // 插值
  .alt($r('app.media.placeholder'))  // 加载占位
  .onComplete(() => {})            // 加载完成回调
  .onError(() => {})               // 加载失败回调
  .borderRadius(8)
```

### Button

```typescript
Button('确定')
  .type(ButtonType.Capsule)      // Capsule/Circle/Normal
  .backgroundColor('#007DFF')
  .fontColor(Color.White)
  .fontSize(16)
  .width('80%')
  .height(44)
  .borderRadius(22)
  .onClick(() => { /* 处理点击 */ })
  .enabled(true)
  .loadingStyle({                // 加载状态
    isLoading: this.isSubmitting,
    spinnerColor: Color.White
  })
```

### TextInput

```typescript
TextInput({ placeholder: '请输入用户名' })
  .type(InputType.Normal)        // Normal/Password/Email/Number
  .maxLength(20)
  .fontSize(16)
  .onChange((value) => {
    this.username = value
  })
  .onSubmit(() => { /* 回车提交 */ })
  .caretColor('#007DFF')
  .showPasswordIcon(true)         // 密码模式显示切换图标
```

### TextArea

```typescript
TextArea({ placeholder: '请输入内容' })
  .maxLength(500)
  .height(120)
  .onChange((value) => { this.content = value })
```

### Toggle

```typescript
Toggle({ type: ToggleType.Switch, isOn: this.isEnabled })
  .onChange((isOn) => { this.isEnabled = isOn })
  .selectedColor('#007DFF')
```

### Progress

```typescript
Progress({ value: 50, total: 100, type: ProgressType.Linear })
  .width('100%')
  .color('#007DFF')
```

### Slider

```typescript
Slider({
  value: this.volume,
  min: 0,
  max: 100,
  step: 1,
  style: SliderStyle.OutSet
})
  .blockColor('#007DFF')
  .trackColor('#ddd')
  .onChange((value) => { this.volume = value })
```

## 4. 弹窗与对话框

### AlertDialog（API 12+ 推荐）

> ⚠️ `AlertDialog.show()` 已废弃，改用 `this.getUIContext().showAlertDialog()`

```typescript
this.getUIContext().showAlertDialog({
  title: '提示',
  message: '确认删除此条记录？',
  buttons: [
    { value: '取消', action: () => {} },
    { value: '确认', action: () => { /* 执行删除 */ } }
  ]
})
```

### ActionSheet（API 12+ 推荐）

> ⚠️ `ActionSheet.show()` 已废弃，改用 `this.getUIContext().showActionSheet()`

```typescript
this.getUIContext().showActionSheet({
  title: '选择操作',
  message: '请选择要执行的操作',
  sheets: [
    { title: '拍照', action: () => {} },
    { title: '从相册选择', action: () => {} },
    { title: '取消', action: () => {} }
  ]
})
```

> ⚠️ 注意：`showActionSheet` 默认 `backgroundBlurStyle: BlurStyle.COMPONENT_ULTRA_THICK`，会与 `backgroundColor` 叠加产生异常效果。如需自定义背景色，必须同时设置：
> ```typescript
> this.getUIContext().showActionSheet({
>   backgroundBlurStyle: BlurStyle.NONE,
>   backgroundColor: '#faf8f2',
>   // ...
> })
> ```

### CustomDialog

> ⚠️ 新版 ArkTS 编译器要求 `controller` 必须声明为可选（`controller?:`），且关闭弹窗需用 `onClose` 回调，避免自引用 TDZ 错误。

```typescript
@CustomDialog
struct MyDialog {
  controller?: CustomDialogController;
  onClose: () => void = () => {};

  build() {
    Column() {
      Text('自定义对话框')
      Button('关闭').onClick(() => { this.onClose(); })
    }
    .padding(20)
  }
}

// 调用（注意显式类型注解 + 箭头函数函数体语法）
const ctrl: CustomDialogController = new CustomDialogController({
  builder: MyDialog({
    onClose: () => { ctrl.close(); }
  }),
  autoCancel: true,
  customStyle: true,
});
ctrl.open();
```

### Toast（API 12+ 推荐）

> ⚠️ 建议使用 `PromptAction` 实例的 `showToast()` 方法：

```typescript
import { PromptAction } from '@kit.ArkUI';

// 在组件中
private promptAction: PromptAction = this.getUIContext().getPromptAction();

// 调用
this.promptAction.showToast({
  message: '操作成功',
  duration: 2000
})
```

## 5. 动画

### 属性动画

```typescript
@State width: number = 100

Column() {
  Text('动画')
    .width(this.width)
    .animation({ duration: 300, curve: Curve.EaseInOut })
}
Button('展开').onClick(() => { this.width = 300 })
```

### 显式动画

```typescript
animateTo({ duration: 300, curve: Curve.EaseInOut }, () => {
  this.width = 300
})
```

### 转场动画

```typescript
if (this.showDetail) {
  Text('详情')
    .transition({ type: TransitionType.Insert, opacity: 0 })
    .transition({ type: TransitionType.Delete, opacity: 0 })
}
```

## 6. 手势

```typescript
Text('可拖动')
  .gesture(
    PanGesture()
      .onActionStart(() => {})
      .onActionUpdate((event) => {
        this.offsetX = event.offsetX
        this.offsetY = event.offsetY
      })
      .onActionEnd(() => {})
  )
  .gesture(
    TapGesture({ count: 2 })
      .onAction(() => { /* 双击 */ })
  )
  .gesture(
    LongPressGesture()
      .onAction(() => { /* 长按 */ })
  )
  .gesture(
    PinchGesture()
      .onActionStart(() => {})
      .onActionUpdate((event) => { this.scale = event.scale })
  )
```
