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

**触发条件**：使用了 `any` 或 `unknown` 类型。

**错误写法**：
```typescript
let data: any = JSON.parse(str)
function handle(val: unknown) { }
```

**正确写法**：使用明确的类型。
```typescript
interface ConfigData {
  version: number
  name: string
}
let data: ConfigData = JSON.parse(str) as ConfigData

// 宁可多加类型定义，也不要用 any
```

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

## 12. arkts-no-any-unknown — `as` 类型断言触发 (10605008)

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

## 13. 漏导类型 — Cannot find name

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

## 14. 枚举 vs 联合类型

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
| `JSON.parse(JSON.stringify(x))` | 手写 `cloneXxx()` 函数 |
| `array.filter(fn)` / `array.map(fn)` | for 循环 |
| `enum X { ... }` | `type X = 'a' \| 'b'` |
| `import { Canvas } from '@kit.ArkUI'` | 不 import，组件内直接使用 |

---

## 编译器错误信息速查

| 错误码 | 关键字 | 一句话说明 |
|--------|--------|-----------|
| 10605040 | `arkts-no-obj-literals-as-types` | 禁止对象字面量做类型 → 定义 interface |
| 10605038 | `arkts-no-untyped-obj-literals` | 禁止无类型对象字面量 → 加类型注解 |
| 10605099 | `arkts-no-spread` | 禁止 spread → 逐字段拷贝 |
| 10605008 | `arkts-no-any-unknown` | 禁止 any/unknown（含 `as` 断言） → 明确类型 + 类型注解 |
| 10605029 | `arkts-no-props-by-index` | 禁止索引访问 → if/switch 展开 |
| 10311006 | `is not exported from Kit` | 导入不存在 → 检查 API 来源 |
| 10505001 | `; expected` / `Declaration expected` | 语法糖不支持 → 可能是箭头函数/Builder 调用方式 |
| — | `Cannot find name 'X'` | 漏导类型 → 在 import 中显式添加 |

---

> **核心原则**：ArkTS 要求一切类型在编译时完全确定，不允许任何形式的"运行时动态类型"。写代码时想象编译器在逐行检查"这个表达式的类型我从 interface 定义中能找到吗？"——找不到就报错。
