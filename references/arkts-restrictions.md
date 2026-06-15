# ArkTS 严格限制与错误对照表

> 本文档记录 TS/JS → ArkTS 编译时最常见的错误及修复方式。ArkTS 是 TypeScript 的严格子集，禁止了大量 TS 常见写法。
> 每个错误附编译错误码、触发条件、错误示例和正确写法。

---

## 1. arkts-no-obj-literals-as-types (10605040) — 禁止对象字面量做类型声明

**触发条件**：在类型注解位置使用了对象字面量（如 `{ x: number; y: number }`）。

**错误写法**：
```typescript
function getPosition(): { x: number; y: number } {
  return { x: 10, y: 20 }
}

let pos: { x: number; y: number } = { x: 10, y: 20 }
```

**正确写法**：必须先定义 interface 或 class，然后用它做类型。
```typescript
interface Position {
  x: number
  y: number
}

function getPosition(): Position {
  return { x: 10, y: 20 }
}

let pos: Position = { x: 10, y: 20 }
```

> ⚠️ **这是 ArkTS 最常见的错误类型**，几乎所有"内联类型"都要改为命名接口。

---

## 2. arkts-no-untyped-obj-literals (10605038) — 禁止无类型对象字面量

**触发条件**：创建对象字面量时，编译器无法推断其对应的类型。

**错误写法**：
```typescript
// 对象字面量没有匹配任何已知 interface/class
const config = { enabled: true, w: 15, h: 28 }

// 函数返回对象字面量但未声明返回类型为 interface
return { id: 1, wall: 'left', offset: 100 }
```

**正确写法**：
```typescript
// 方案 A：对象字面量赋值给已声明类型的变量
const config: Corner = { enabled: true, w: 15, h: 28 }

// 方案 B：函数声明了明确的返回类型（必须是 interface）
function createOpening(): Opening {
  return { id: 1, wall: 'left', offset: 100, width: 80, type: 'door', direction: 'left' }
}
```

> 💡 **经验法则**：ArkTS 中任何 `{ ... }` 直接写出来的对象字面量，都必须能让编译器找到对应的 interface/class。如果找不到，报此错误。

---

## 3. arkts-no-spread (10605099) — 禁止 spread 操作符

**触发条件**：使用 `...` 展开数组或对象。

**错误写法**：
```typescript
// 对象 spread
const corners: Corners = {
  bl: { ...DEFAULT_CORNER },  // ❌
  br: { ...DEFAULT_CORNER },  // ❌
}

// 数组 spread
const merged = [...arr1, ...arr2]

// rest 参数
function foo(...args: number[]) { }
```

**正确写法**：逐字段拷贝，或写 clone 函数。
```typescript
// 逐字段拷贝
const corners: Corners = {
  bl: { enabled: DEFAULT_CORNER.enabled, w: DEFAULT_CORNER.w, h: DEFAULT_CORNER.h },
  br: { enabled: DEFAULT_CORNER.enabled, w: DEFAULT_CORNER.w, h: DEFAULT_CORNER.h },
}

// 或封装 clone 函数
function cloneCorner(c: Corner): Corner {
  return { enabled: c.enabled, w: c.w, h: c.h }
}

const corners: Corners = {
  bl: cloneCorner(DEFAULT_CORNER),
  br: cloneCorner(DEFAULT_CORNER),
}

// 数组合并用 push 循环
const merged: number[] = []
for (let i = 0; i < arr1.length; i++) { merged.push(arr1[i]) }
for (let i = 0; i < arr2.length; i++) { merged.push(arr2[i]) }
```

---

## 4. arkts-no-any-unknown (10605008) — 禁止 any/unknown

**触发条件**：使用了 `any` 或 `unknown` 类型（**包括函数参数、变量声明、`JSON.parse` 返回值未显式接收**）。

**错误写法**：
```typescript
let data: any = JSON.parse(str)          // ❌ any
function handle(val: unknown) { }        // ❌ unknown 参数
function num(val: unknown, def: number)  // ❌ unknown 参数
```

**正确写法 A — 类型已知，直接断言**：
```typescript
interface ConfigData {
  version: number
  name: string
}
const data: ConfigData = JSON.parse(str) as ConfigData
```

**正确写法 B — JSON 反序列化 + 逐字段安全读取（最常用）**：

> 场景：从 Preferences / 剪贴板反序列化 JSON，结构不完全可控时。

```typescript
// 1. 用 Object 接收（ArkTS 允许 Object，禁止 unknown/any）
const parsed: Object = JSON.parse(jsonStr) as Object;
if (parsed === null || typeof parsed !== 'object') { return null; }

// 2. 强转为 Record<string, Object>（允许按字符串 key 访问）
const record = parsed as Record<string, Object>;

// 3. 定义安全读取辅助函数（参数类型用 Object，不用 unknown）
function num(val: Object, fallback: number): number {
  if (typeof val === 'number') {
    const n = val as number;
    if (!isNaN(n) && isFinite(n)) return n;
  }
  return fallback;
}
function str(val: Object, fallback: string): string {
  if (typeof val === 'string') return val as string;
  return fallback;
}
function bool(val: Object, fallback: boolean): boolean {
  if (typeof val === 'boolean') return val as boolean;
  return fallback;
}

// 4. 使用
const id = str(record['id'], '');
const width = num(record['width'], 380);

// 5. 数组也需要显式类型
const openingsRaw: Object = record['openings'];
if (Array.isArray(openingsRaw)) {
  const arr = openingsRaw as Object[];
  for (let i = 0; i < arr.length; i++) {
    openings.push(plainToOpening(arr[i]));  // arr[i] 是 Object，合法
  }
}
```

**关键规则**：
- `JSON.parse()` 返回值必须 `as Object` 显式接收
- 安全辅助函数参数用 `Object`，不用 `unknown`
- `getSchemeNames()` 等解析数组时，`parsed` 也要 `as Object` 后 `as Object[]`

---

## 5. arkts-no-props-by-index (10605029) — 禁止索引访问

**触发条件**：使用 `obj[key]` 方式访问属性（即使 key 是字符串字面量 union 也不行）。

**错误写法**：
```typescript
type Wall = 'left' | 'right' | 'top' | 'bottom'

function getCorner(corners: Corners, wall: Wall): Corner {
  return corners[wall]  // ❌ 索引访问
}

// 即使是明确的字符串也不行
const c = corners['left']  // ❌
```

**正确写法**：用 if/else 或 switch 展开所有分支。
```typescript
function getCorner(corners: Corners, wall: Wall): Corner {
  if (wall === 'left') return corners.left
  if (wall === 'right') return corners.right
  if (wall === 'top') return corners.top
  return corners.bottom
}

// 或用封装好的 getter/setter
class CornersWrapper {
  bl: Corner; br: Corner; tl: Corner; tr: Corner

  get(wall: Wall): Corner {
    if (wall === 'left') return this.bl
    if (wall === 'right') return this.br
    if (wall === 'top') return this.tl
    return this.tr
  }
}
```

> 💡 **模式**：遇到需要按 key 访问的场景，写一个专用的 getter/setter 函数，内部用 if/switch 展开所有可能的 key。如果 key 数量很多（>10），考虑用 `Map<K, V>` 替代。

---

## 6. Record<K, V> 被禁止 — 改用数组 + 查找函数

**触发条件**：使用 `Record<string, T>` 或类似泛型对象类型。

`Record<K, V>` 本质上是 `{ [key: K]: V }`，它是对象字面量类型的语法糖，ArkTS 禁止此模式。

**错误写法**：
```typescript
// ❌ Record 间接引入了对象字面量类型
type SchemeMap = Record<string, Scheme>

let schemes: SchemeMap = {}
schemes['abc'] = newScheme  // ❌ 索引访问也伴随报错
```

**正确写法**：用数组 + 查找/插入函数。
```typescript
interface SchemeEntry {
  id: string
  data: Scheme
}

let schemes: SchemeEntry[] = []

// 查找
function findScheme(id: string): SchemeEntry | undefined {
  for (let i = 0; i < schemes.length; i++) {
    if (schemes[i].id === id) return schemes[i]
  }
  return undefined
}

// 插入
function insertScheme(entry: SchemeEntry): void {
  schemes.push(entry)
}

// 删除
function deleteScheme(id: string): void {
  const filtered: SchemeEntry[] = []
  for (let i = 0; i < schemes.length; i++) {
    if (schemes[i].id !== id) filtered.push(schemes[i])
  }
  schemes = filtered
}
```

---

## 7. 可选属性被禁止 — 改为必填 + 默认值

**触发条件**：interface 或 class 中有 `field?: Type` 可选属性。

**错误写法**：
```typescript
interface Opening {
  id: number
  wall: string
  offset: number
  width: number
  type: 'door' | 'window'
  direction?: 'left' | 'right'  // ❌ 可选属性
}
```

**正确写法**：改为必填，给默认值。
```typescript
interface Opening {
  id: number
  wall: string
  offset: number
  width: number
  type: 'door' | 'window'
  direction: 'left' | 'right'  // 必填
}

// window 不需要 direction，但也要填一个默认值
const sampleWindow: Opening = {
  id: 3, wall: 'left', offset: 180, width: 142,
  type: 'window', direction: 'left'  // 给默认值
}
```

---

## 8. Canvas / CanvasRenderingContext2D 导入错误 (10311006)

**触发条件**：试图从 `@kit.ArkUI` 导入 Canvas 或 CanvasRenderingContext2D。

**错误写法**：
```typescript
import { Canvas, CanvasRenderingContext2D } from '@kit.ArkUI'  // ❌ 不存在
```

**正确写法**：不在 import 中引入。在 ArkUI Canvas 组件的 `onReady` 回调中，参数直接就是 Canvas 类型，其 context 就是 CanvasRenderingContext2D。

```typescript
// 正确：组件内部直接使用，不 import
@Component
struct MyCanvas {
  settings: RenderingContextSettings = new RenderingContextSettings(true)
  ctx: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.settings)

  build() {
    Canvas(this.ctx)
      .onReady(() => {
        // this.ctx 可直接使用 Canvas 2D API
        this.ctx.fillRect(0, 0, 100, 100)
      })
  }
}
```

---

## 9. @Builder 调用限制

**触发条件**：在组件属性赋值位置调用 `@Builder` 函数。

**错误写法**：
```typescript
@Builder function myContent() { Text('hello') }

build() {
  Column() {
    // ❌ 不能在属性位置调用
    Button('click')
      .content(myContent())  // 不存在的 API
  }
}
```

**正确写法**：`@Builder` 只能作为子组件直接在 `build()` 中调用。

```typescript
@Builder function myContent() { Text('hello') }

build() {
  Column() {
    myContent()  // ✅ 直接调用
    // 不能用 .content(myContent()) 这种属性赋值方式
  }
}
```

> 💡 **替代方案**：如果需要在属性位置传递 UI 片段，用 `@BuilderParam` 装饰器。

---

## 10. 箭头函数限制 — .filter/.map 可能失败

**触发条件**：在 `.filter()`、`.map()` 等数组方法中使用箭头函数。

某些 ArkTS 版本对闭包中的箭头函数支持有限制。

**推荐写法**：用 for 循环替代。
```typescript
// 推荐代替 filter
const result: Item[] = []
for (let i = 0; i < list.length; i++) {
  if (list[i].active) {
    result.push(list[i])
  }
}

// 推荐代替 map
const mapped: string[] = []
for (let i = 0; i < list.length; i++) {
  mapped.push(list[i].name)
}
```

---

## 11. JSON 深度克隆 — 需手写 clone 函数

**触发条件**：用 `JSON.parse(JSON.stringify(obj))` 做深拷贝——语法合法但可能遇到 ArkTS 的类型推断问题。

**推荐写法**：为每个复杂类型手写 clone 函数。
```typescript
function cloneRoom(room: Room): Room {
  return {
    width: room.width,
    length: room.length,
    corners: {
      bl: cloneCorner(room.corners.bl),
      br: cloneCorner(room.corners.br),
      tl: cloneCorner(room.corners.tl),
      tr: cloneCorner(room.corners.tr),
    }
  }
}

function cloneOpenings(list: Opening[]): Opening[] {
  const result: Opening[] = []
  for (let i = 0; i < list.length; i++) {
    const o = list[i]
    result.push({ id: o.id, wall: o.wall, offset: o.offset, width: o.width, type: o.type, direction: o.direction })
  }
  return result
}
```

---

## 12. @State 观察范围与 @Observed/@ObjectLink（核心！）

### @State 能观察什么

| @State 变量类型 | 能观察 | 不能观察 |
|----------------|--------|----------|
| 基本类型 | 值变化 | — |
| 对象（第一层） | 属性赋值 `this.obj = newObj` | 嵌套属性 `this.obj.field = x` |
| 数组 | push / splice / shift / unshift / 整体赋值 | **数组项对象的属性变化** `this.arr[0].x = x` |

**关键结论**：
- ✅ `@State` 数组 **可以** 响应 `.push()` / `.splice()` 等数组方法
- ❌ `@State` **不能** 响应嵌套对象的属性变化（如 `room.width = 400`、`furniture[0].x = 100`）

### @Observed + @ObjectLink 解决嵌套对象属性变化

**适用场景**：当 `@State` 对象的属性本身也是对象（嵌套对象），需要观察其内部属性变化时。

**用法**：
1. `@Observed` 装饰 **class**（不是 interface！）
2. `@ObjectLink` 装饰 **子组件** 中的变量
3. 父组件 `@State` 持有数组/对象，传递子项给子组件

```typescript
// 1. @Observed 装饰类（必须是 class，不能是 interface）
@Observed
class Furniture {
  id: number = 0
  name: string = ''
  x: number = 0
  y: number = 0
  // ... 其他字段

  constructor(id: number, name: string, x: number, y: number /* ... */) {
    this.id = id
    this.name = name
    this.x = x
    this.y = y
    // ...
  }
}

// 2. 子组件用 @ObjectLink 接收嵌套对象
@Component
struct FurnitureListItem {
  @ObjectLink item: Furniture  // 接收 @Observed 实例

  build() {
    Row() {
      Text(this.item.name + ' ' + this.item.x)  // 自动响应 item 属性变化
    }
  }
}

// 3. 父组件用 @State 持有数组
@Entry
@Component
struct Parent {
  @State furniture: Furniture[] = []

  build() {
    Column() {
      ForEach(this.furniture, (f: Furniture) => {
        FurnitureListItem({ item: f })  // 传递给 @ObjectLink
      }, (f: Furniture) => f.id.toString())
    }
  }
}
```

### @Observed/@ObjectLink 注意事项

| 规则 | 说明 |
|------|------|
| `@Observed` 只能装饰 class | interface 不行，必须转为 class + constructor |
| `@ObjectLink` 不能用于 @Entry 组件 | 只能在子组件中使用 |
| `@ObjectLink` 变量不能重新赋值 | 不能 `this.item = newItem`，只能改属性 `this.item.x = 1` |
| `@ObjectLink` 不支持基本类型 | 需要 `@Prop` 替代 |
| 创建实例必须用 `new` | 不能用对象字面量 `{ id: 1, ... }` |
| 嵌套属性也需 `@Observed` | 如果 `Room.corners` 的 `Corners` 也需要深层观察，`Corners` 也必须 `@Observed` |

### 完整的响应式分层策略

```
@State（顶层）  → 观察：属性赋值 + 数组 push/splice
  ↓ 传递给子组件
@ObjectLink（子组件）→ 观察：@Observed 实例的属性变化
  ↓ 如果属性也是 @Observed
@ObjectLink（孙组件）→ 继续观察更深层变化
```

> 💡 **口诀**：`@State` 管浅层（数组增删、对象替换），`@Observed` + `@ObjectLink` 管深层（嵌套属性变化）。

---

## 13. @State 对象嵌套更新 — 正确方案

**触发条件**：直接修改 `@State` 对象的深层属性（如 `this.state.room.width = 400`），UI 不刷新。

### 方案 A（推荐）：@Observed + @ObjectLink

```typescript
@Observed
class Room {
  width: number = 380
  length: number = 460
  corners: Corners = new Corners(...)
  constructor(width: number, length: number, corners: Corners) { ... }
}

// 在父组件中直接修改属性
this.state.room.width = 400  // @Observed 代理检测到变化，通知 @ObjectLink 子组件
```

### 方案 B（备选）：整体替换对象

```typescript
// 不使用 @Observed 时，整体替换
this.state.room = new Room(400, this.state.room.length, this.state.room.corners)
```

> 方案 A 更符合官方推荐，代码更简洁，不需要每次都重建对象。

---

## 14. arkts-no-any-unknown — `as` 类型断言触发 (10605008)

**触发条件**：在变量赋值时使用 `as Type` 类型断言（即使目标类型是明确定义的联合类型）。

ArkTS 编译器认为 `as` 断言引入了不安全的类型转换，等价于 `any` 赋值。

**错误写法**：
```typescript
const type = (this.openingType === 0) ? 'door' as OpeningType : 'window' as OpeningType
```

**正确写法**：用显式类型注解代替 `as` 断言。
```typescript
const type: OpeningType = (this.openingType === 0) ? 'door' : 'window'
```

> 💡 **原则**：ArkTS 中永远不要用 `as Type`——全部改为 `const x: Type = ...` 或 `const x: Type[] = [...]`。类型注解要放在变量声明侧，不要放在值表达式中。

---

## 15. 漏导类型 — Cannot find name

**触发条件**：在文件中使用了某个类型（如 `OpeningType`）但没有在 `import type { ... }` 中显式导入。

**错误写法**：
```typescript
// DesignTypes.ets 导出了 OpeningType
export type OpeningType = 'door' | 'window'

// Index.ets 漏导了 OpeningType
import type { DesignState, Wall } from '../core/DesignTypes'

// 但代码中使用了
const type: OpeningType = 'door'  // ❌ Cannot find name 'OpeningType'
```

**正确写法**：在 import 列表中显式加上所有用到的类型名。
```typescript
import type { DesignState, Wall, OpeningType } from '../core/DesignTypes'
```

> 💡 **经验法则**：每次写完代码后搜索所有类型引用，确认它们都在 import 列表中。ArkTS 不会像 TS 那样自动补全或推断跨文件的类型引用。

---

## 16. 枚举 vs 联合类型

**推荐**：用 `type` 联合类型替代 `enum`，ArkTS 对 enum 支持有限。

```typescript
// ❌ 不推荐
enum WallType { door, window }

// ✅ 推荐
type OpeningType = 'door' | 'window'
type Wall = 'left' | 'right' | 'top' | 'bottom'
type DoorDirection = 'left' | 'right'
```

---

## TS → ArkTS 翻译速查表

| TypeScript 写法 | ArkTS 替代方案 |
|----------------|---------------|
| `{ x: number; y: number }`（内联类型） | 先定义 `interface Pos { x: number; y: number }` |
| `Record<string, T>` | `Entry[]` 数组 + 查找函数 |
| `const x = { a: 1, b: 2 }`（无类型对象） | 加类型注解 `const x: MyType = { ... }` |
| `{ ...obj }` / `[...arr]` | 逐字段拷贝 / clone 函数 / push 循环 |
| `obj[key]` | if/switch 展开 / getter-setter 函数 |
| `any` / `unknown` | 明确的 interface/class 类型 |
| `'val' as Type`（as 断言） | `const x: Type = 'val'` 类型注解 |
| `field?: Type`（可选） | 必填 `field: Type` + 默认值 |
| 嵌套对象属性变化 | `@Observed` class + 子组件 `@ObjectLink` |
| `interface` 需要深层观察 | 改为 `@Observed class` + constructor |
| `JSON.parse(JSON.stringify(x))` | 手写 `cloneXxx()` 函数 |
| `private scale = 1`（与基类冲突） | 改名 `drawScale` 等不冲突的词 |
| `AlertDialog.show()` / `ActionSheet.show()` | `this.getUIContext().showAlertDialog()` / `.showActionSheet()` |
| `array.filter(fn)` / `array.map(fn)` | for 循环 |
| `enum X { ... }` | `type X = 'a' \| 'b'` |
| `import { Canvas } from '@kit.ArkUI'` | 不 import，组件内直接使用 |
| `AlertDialog.show({...})` / `ActionSheet.show({...})` | `this.getUIContext().showAlertDialog({...})` / `this.getUIContext().showActionSheet({...})` |
| 给 ActionSheet 加 `backgroundColor` 颜色异常 | 同时设置 `backgroundBlurStyle: BlurStyle.NONE`，或改用 `@CustomDialog` |
| 需要美观自定义弹窗 | `@CustomDialog` + `CustomDialogController`，设 `customStyle: true` |\n| `@CustomDialog` 中 `controller: CustomDialogController = new ...` 自引用初始化 | ⚠️ **会导致 undefined**！改为 `controller?:` + `onClose` 回调关闭弹窗 |
| `const ctrl = new ...` 无类型注解 | 推断为 any → 加 `const ctrl: CustomDialogController = ...` |
| builder 内箭头函数用表达式 `() => expr` | 返回类型推断受限 → 改为 `{ }` 函数体 |

---

## 17. 弹窗选择决策

**ActionSheet 局限性：**
- 默认 `backgroundBlurStyle: BlurStyle.COMPONENT_ULTRA_THICK` 与 `backgroundColor` 叠加导致颜色异常
- 即使修复模糊冲突，样式定制能力仍有限
- **推荐：需要自定义样式的操作菜单 → 使用 `@CustomDialog`**

**CustomDialog 标准写法（v1.5 最终版）：**

```typescript
@CustomDialog
struct MyActionDialog {
  // ⚠️ controller 可选声明（满足编译器两个约束）
  // 约束1：必须有 CustomDialogController 类型属性
  // 约束2：属性必须有运行时无关默认值 → 用 ? 语法
  controller?: CustomDialogController;
  title: string = '';
  onAction: () => void = () => {};
  onClose: () => void = () => {};   // ⚠️ 关闭弹窗用回调，不用 this.controller

  build() {
    Column() {
      Text(this.title).fontSize(16)
      Button('操作').onClick(() => { this.onClose(); this.onAction(); })
    }
    .backgroundColor(Color.White).borderRadius(14)
  }
}

// 父组件中打开
showDialog(): void {
  // ⚠️ 必须显式类型注解，否则推断为 any
  const ctrl: CustomDialogController = new CustomDialogController({
    builder: MyActionDialog({
      title: '标题',
      onAction: () => { /* 逻辑 */ },
      // ⚠️ 不能传 controller: ctrl（TDZ 错误）
      // ⚠️ 箭头函数必须用 {} 函数体
      onClose: () => { ctrl.close(); },
    }),
    alignment: DialogAlignment.Bottom,
    customStyle: true,
    autoCancel: true,
    maskColor: 'rgba(0,0,0,0.4)',
  });
  ctrl.open();
}
```

### 为什么必须用 `controller?:` + `onClose` 回调？

新版 ArkTS 编译器有多个互斥约束：

| 写法 | 结果 |
|------|------|
| `controller: CustomDialogController;` 无默认值 | ❌ "runtime-independent default value should be set" |
| `controller = new ...` 自引用 | ❌ 运行时 undefined，close() TypeError |
| 去掉 `controller` 属性 | ❌ "@CustomDialog must contain a property of the CustomDialogController type" |
| **`controller?:` + `onClose` 回调** | ✅ 通过编译 + 运行时正常 |

### 另外两个编译陷阱

| 写法 | 错误 | 正确写法 |
|------|------|---------|
| `const ctrl = new ...` 无类型 | 推断为 any → `arkts-no-any-unknown` | **`const ctrl: CustomDialogController = ...`** |
| `onClose: () => ctrl.close()` 表达式箭头函数 | `arkts-no-implicit-return-types` | **`onClose: () => { ctrl.close(); }`** 函数体 |
| builder 中传 `controller: ctrl` | TDZ "used before its declaration" | 不传，改用 `onClose` 回调 |

---

## 18. CustomComponent 属性名冲突 — 不能用内置方法名做变量名

**触发条件**：在 `@Component` struct 中声明 `private`/`@State` 变量时，使用了 ArkUI `CustomComponent` 基类的内置属性方法名。

ArkUI 的 `CustomComponent` 基类自带大量属性方法（如 `.scale()`, `.opacity()`, `.rotate()`, `.translate()`, `.width()`, `.height()` 等），这些方法名同时也是属性名。如果你的变量名和它们同名，编译器会报类型冲突：

```
Property 'scale' in type 'Index' is not assignable to the same property in base type 'CustomComponent'.
Type 'number' is not assignable to type '{ (value: ScaleOptions): CommonAttribute; ... }'.
```

**错误写法**：
```typescript
@Entry
@Component
struct Index {
  private scale: number = 1  // ❌ 与 CustomComponent.scale 冲突
}
```

**正确写法**：使用不会冲突的命名（加前缀或换词）。
```typescript
@Entry
@Component
struct Index {
  private drawScale: number = 1  // ✅ 不冲突
}
```

> 💡 **已知冲突词列表**：scale, opacity, rotate, translate, width, height, margin, padding, border, offset, position, flex, align, visibility, enabled, focus, gesture, response, transition, animation, shadow, blur, grayscale, brightness, saturate, invert, sepia, hueRotate, clip, mask, overlay, zIndex, id, key, grid, gridSpan, gridOffset, useSize, constraintSize, aspectRatio, decoration, backgroundColor, backgroundImage, backgroundBlur, clipShape, shape, ripple, hover, pressed, selected, checked, style, draggable, drag, drop, bind, on, gestureGroup, priority, parallel, sequence, custom, hitTest, touch, mouse, key, focus, hover, appear, disappear, area, size, layout, draw, render, measure, align, alignSelf, alignSelf, flexGrow, flexShrink, flexBasis, display, visibility, aspectRatio

---

## 19. AlertDialog.show / ActionSheet.show 已废弃 — 改用 UIContext 方法

**触发条件**：使用 `AlertDialog.show()` 或 `ActionSheet.show()` 静态方法弹出对话框。

从 HarmonyOS Next API 12+ 起，这些静态方法被标记为 deprecated（废弃），编译时会输出警告：

```
ArkTS:WARN 'show' has been deprecated.
```

**废弃写法**：
```typescript
AlertDialog.show({
  title: '确认',
  message: '确定吗？',
  primaryButton: { value: '取消', action: () => {} },
  secondaryButton: { value: '确认', action: () => { /* ... */ } }
});

ActionSheet.show({
  title: '操作',
  sheets: [ { title: '选项1', action: () => {} } ],
  confirm: { value: '取消', action: () => {} }
});
```

**正确写法**：通过 `this.getUIContext()` 获取 UIContext 实例，再调用对应方法：
```typescript
// AlertDialog → showAlertDialog，buttons 数组替代 primaryButton/secondaryButton
this.getUIContext().showAlertDialog({
  title: '确认',
  message: '确定吗？',
  buttons: [
    { value: '取消', action: () => {} },
    { value: '确认', action: () => { /* ... */ } }
  ]
});

// ActionSheet → showActionSheet，参数结构不变
this.getUIContext().showActionSheet({
  title: '操作',
  sheets: [ { title: '选项1', action: () => {} } ],
  confirm: { value: '取消', action: () => {} }
});
```

> ⚠️ **关键差异**：
> - `showAlertDialog` 的按钮配置从 `primaryButton/secondaryButton` 改为 `buttons[]` 数组
> - `showActionSheet` 的参数结构与旧 API 一致，只是调用方式变了
> - 必须在组件内部调用 `this.getUIContext()`，不能在异步回调深处或后台任务中使用

---

## 编译器错误信息速查

| 错误码 | 关键字 | 一句话说明 |
|--------|--------|-----------|
| 10605040 | `arkts-no-obj-literals-as-types` | 禁止对象字面量做类型 → 定义 interface |
| 10605038 | `arkts-no-untyped-obj-literals` | 禁止无类型对象字面量 → 加类型注解 |
| 10605099 | `arkts-no-spread` | 禁止 spread → 逐字段拷贝 |
| 10605008 | `arkts-no-any-unknown` | 禁止 any/unknown（含函数参数） → **用 `Object` 替代** `unknown`，`JSON.parse()` 必须 `as Object` |
| 10605029 | `arkts-no-props-by-index` | 禁止索引访问 → if/switch 展开 |
| 10311006 | `is not exported from Kit` | 导入不存在 → 检查 API 来源 |
| 10505001 | `; expected` / `Declaration expected` | 语法糖不支持 → 可能是箭头函数/Builder 调用方式 |
| — | `a valid, runtime-independent default value should be set for it` | CustomDialog controller 缺少默认值 → 改为 `controller?: CustomDialogController` |
| — | `@CustomDialog must contain a property of the CustomDialogController type` | 缺少 controller 属性 → 必须声明 `controller?: CustomDialogController` |
| 10605008 | `Use explicit types instead of "any", "unknown"` | 变量无类型注解推断为 any → 加显式类型如 `const ctrl: CustomDialogController = ...` |
| 10605090 | `Function return type inference is limited` | 箭头函数用表达式语法 → 改为 `{ }` 函数体：`() => { expr; }` |
| — | `Block-scoped variable 'ctrl' used before its declaration` | builder 中传 `controller: ctrl`（TDZ）→ 改用 `onClose` 回调 |
| — | `Cannot find name 'X'` | 漏导类型 → 在 import 中显式添加 |

---

> **核心原则**：ArkTS 要求一切类型在编译时完全确定，不允许任何形式的"运行时动态类型"。写代码时想象编译器在逐行检查"这个表达式的类型我从 interface 定义中能找到吗？"——找不到就报错。
