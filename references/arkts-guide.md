# ArkTS 语言详细语法参考

> 来源：https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-get-started
> 优先以官方文档为准，本文档作为快速参考。

## 1. ArkTS 与 TypeScript 的差异

ArkTS 基于 TypeScript，但做了以下限制：

- **禁止使用 any 类型**：必须显式类型注解
- **禁止使用 unknown**：同上
- **禁止使用 as 类型断言的某些场景**：受限
- **强制严格空值检查**：null / undefined 必须显式处理
- **不支持 eval / new Function**：安全限制
- **不支持原型链操作**：不支持 `__proto__`、`Object.setPrototypeOf` 等

## 2. 基本数据类型

```typescript
// 基础类型
let isDone: boolean = false
let count: number = 10
let name: string = 'HarmonyOS'

// 数组
let list: number[] = [1, 2, 3]
let list2: Array<number> = [1, 2, 3]

// 联合类型
let value: string | number = 'hello'

// 枚举
enum Direction { Up, Down, Left, Right }
let dir: Direction = Direction.Up
```

## 3. 接口与类

```typescript
// 接口
interface Person {
  name: string
  age: number
  greet(): string
}

// 类
class Student implements Person {
  name: string
  age: number

  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }

  greet(): string {
    return `Hello, I'm ${this.name}`
  }
}
```

## 4. 泛型

```typescript
function identity<T>(arg: T): T {
  return arg
}

interface Result<T> {
  code: number
  data: T
  message: string
}
```

## 5. 模块导入导出

```typescript
// 导出
export class MyClass { }
export function myFunc() { }
export const MY_CONST = 42

// 导入
import { MyClass, myFunc } from './myModule'
import * as Utils from './utils'
```

## 6. 空值安全

```typescript
// 可空类型
let name: string | null = null

// 非空断言（确认不为空时使用）
let length: number = name!.length

// 可选链
let city: string | undefined = user?.address?.city

// 空值合并
let displayName: string = name ?? '未知'
```

## 7. 装饰器详解

### @Entry

标记页面入口组件，一个页面只能有一个 @Entry。

```typescript
@Entry
@Component
struct MainPage {
  build() { /* ... */ }
}
```

### @Component

标记自定义组件，必须实现 build() 方法。

```typescript
@Component
export struct MyCard {
  build() {
    Column() { /* ... */ }
  }
}
```

### @State

组件内部可变状态，值变化时触发 UI 刷新。支持基本类型、类、数组。

```typescript
@State count: number = 0
@State list: number[] = [1, 2, 3]
@State user: User = new User()
```

### @Prop

父组件单向传递给子组件，子组件可读取但修改不回传。支持基本类型。

```typescript
@Component
struct Child {
  @Prop title: string = ''

  build() {
    Text(this.title)
  }
}
```

### @Link

父子双向数据绑定，子组件修改会同步回父组件。不支持默认值初始化。

```typescript
@Component
struct Child {
  @Link count: number

  build() {
    Button(`+1`).onClick(() => { this.count++ })
  }
}

// 父组件传递
Child({ count: $count })  // $ 语法
```

### @Provide / @Consume

跨层级数据共享，祖先提供，后代消费。

```typescript
// 祖先组件
@Provide theme: string = 'light'

// 后代组件（无需逐层传递）
@Consume theme: string
```

### @Watch

监听状态变化，触发回调。

```typescript
@State @Watch('onCountChange') count: number = 0

onCountChange() {
  console.log(`count changed to ${this.count}`)
}
```

### @Builder

轻量级 UI 复用函数，不支持状态管理。

```typescript
@Builder function MyText(label: string) {
  Text(label).fontSize(16)
}

// 调用
MyText('Hello')
```

### @Extend

扩展原生组件的样式方法。

```typescript
@Extend(Text) function fancyText() {
  .fontSize(20)
  .fontWeight(FontWeight.Bold)
  .fontColor('#333')
}

// 使用
Text('Hello').fancyText()
```

### @Styles

样式复用，只能包含通用属性。

```typescript
@Styles function centerStyle() {
  .width('100%')
  .height('100%')
  .justifyContent(FlexAlign.Center)
  .alignItems(HorizontalAlign.Center)
}
```

## 8. 组件生命周期

```typescript
@Component
struct MyComponent {
  aboutToAppear() {
    // 组件即将出现，在此请求数据
  }

  aboutToDisappear() {
    // 组件即将销毁，在此释放资源
  }

  onPageShow() {
    // 页面每次显示时（仅 @Entry 组件）
  }

  onPageHide() {
    // 页面每次隐藏时（仅 @Entry 组件）
  }

  onBackPress() {
    // 返回键按下时（仅 @Entry 组件）
    // 返回 true 表示消费事件，不再继续传递
    return true
  }

  build() { /* ... */ }
}
```
