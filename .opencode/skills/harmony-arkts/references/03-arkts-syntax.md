# ArkTS 与 TypeScript 基础语法

## 技术演进

| 阶段 | 框架 | 说明 |
|------|------|------|
| HarmonyOS 1-2.x | Java UI / JS UI | 早期框架 |
| HarmonyOS 3.x+ | **ArkTS + ArkUI** | 当前主力框架 |
| HarmonyOS NEXT | **ArkTS (API 12+)** | 严格类型约束 |

## 约束对比

```
JavaScript → TypeScript → ArkTS
```

**关键约束**：
- 强制使用静态类型
- 禁止在运行时改变对象布局
- 限制运算符语义
- 不支持结构化类型 (Structural Typing)
- **对象字面量必须有显式类型声明**（API 12+ 严格模式）

## 变量与常量

```typescript
// 变量
let name: string = 'Jack'
let age: number = 20
let isLogin: boolean = false

// 常量
const PI: number = 3.14
```

## 数组与对象

```typescript
// 数组 - 必须显式声明类型
let nameList: string[] = ['Jack', 'Rose', 'James']

// 接口定义对象结构
interface Person {
  name: string
  age: number
}

let person: Person = {
  name: 'Rose',
  age: 18
}
```

## 函数

```typescript
// 函数声明
function add(x: number, y: number): number {
  return x + y
}

// 可选参数
function say(name?: string) {
  if (name) {
    console.log('Hello', name)
  }
}

// 箭头函数
let add = (x: number, y: number): number => x + y
```

## 类

```typescript
class Person {
  name: string = ''
  age: number = 0

  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }

  speak(): void {
    console.log(`I'm ${this.name}`)
  }
}
```

## ArkTS 严格类型规则（API 12+）

### 规则 1：禁止对象字面量作为类型声明 (arkts-no-obj-literals-as-types)

**错误示例**：
```typescript
// ❌ 错误：使用对象字面量作为类型
type Person = { name: string, age: number }

interface Waypoint {
  coordinates: {        // ❌ 错误：内联对象字面量
    latitude: number
    longitude: number
  }
}
```

**正确示例**：
```typescript
// ✅ 正确：使用 interface 显式定义类型
interface Person {
  name: string
  age: number
}

// ✅ 正确：提取为独立接口
interface Coordinates {
  latitude: number
  longitude: number
}

interface Waypoint {
  coordinates: Coordinates
}
```

### 规则 2：对象字面量必须对应显式声明的类或接口 (arkts-no-untyped-obj-literals)

**错误示例**：
```typescript
// ❌ 错误：对象字面量没有类型声明
const area = {
  pixels: new ArrayBuffer(8),
  offset: 0,
  stride: 8
}

// ❌ 错误：数组中包含无类型对象字面量
const waypoints = [
  { id: 1, name: 'Paris', coordinates: { latitude: 48.8, longitude: 2.3 } },
  { id: 2, name: 'NYC', coordinates: { latitude: 40.7, longitude: -74.0 } }
]
```

**正确示例**：
```typescript
// ✅ 正确：先声明类型，再创建对象
interface Area {
  pixels: ArrayBuffer
  offset: number
  stride: number
}

const area: Area = {
  pixels: new ArrayBuffer(8),
  offset: 0,
  stride: 8
}

// ✅ 正确：为数组元素声明类型
interface Coordinates {
  latitude: number
  longitude: number
}

interface Waypoint {
  id: number
  name: string
  coordinates: Coordinates
}

// 方案1：使用显式类型变量
const wp1: Waypoint = {
  id: 1,
  name: 'Paris',
  coordinates: { latitude: 48.8, longitude: 2.3 }
}

const wp2: Waypoint = {
  id: 2,
  name: 'NYC',
  coordinates: { latitude: 40.7, longitude: -74.0 }
}

const waypoints: Waypoint[] = [wp1, wp2]

// 方案2：使用 Record 类型（键值对对象）
let emitArg: Record<string, number | boolean> = {
  'action': 11,
  'outers': false
}
```

### 规则 3：数组字面量元素必须是可推断类型 (arkts-no-noninferrable-arr-literals)

**错误示例**：
```typescript
// ❌ 错误：数组元素类型不明确
let permissionList = [
  { name: '设备信息', value: '用于分析设备' },
  { name: '麦克风', value: '用于语音' }
]

// ❌ 错误：直接在 @State 中使用数组字面量
@State waypoints: Waypoint[] = [
  { id: 1, name: 'Paris', coordinates: { latitude: 48.8, longitude: 2.3 } }
]
```

**正确示例**：
```typescript
// ✅ 正确：为数组元素声明类型
interface PermissionItem {
  name: string
  value: string
}

let permissionList: PermissionItem[] = [
  { name: '设备信息', value: '用于分析设备' },
  { name: '麦克风', value: '用于语音' }
]

// ✅ 正确：在组件中使用预定义变量
@Component
struct MapComponent {
  private coord1: Coordinates = { latitude: 48.8, longitude: 2.3 }
  private wp1: Waypoint = { id: 1, name: 'Paris', coordinates: this.coord1 }
  
  @State waypoints: Waypoint[] = [this.wp1]
}
```

## ArkTS 类型系统设计原则

1. **编译时确定对象布局**：ArkTS 中对象布局在编译时是确定的，不能在运行时改变
2. **增强类型安全性**：要求显式类型声明，减少运行时类型检查开销
3. **避免行为二义性**：禁止通过对象字面量绕过 class 的构造函数验证
4. **提高代码可维护性**：显式类型声明使代码更易于理解和维护

## 常见错误场景及解决方案

### 场景 1：JSON.parse 返回值类型

```typescript
// ❌ 错误
let data = JSON.parse(jsonString)

// ✅ 正确
interface MyData {
  id: string
  value: number
}
let data: MyData = JSON.parse(jsonString) as MyData
```

### 场景 2：泛型函数调用

```typescript
// ❌ 错误：无法推断泛型类型参数
let y = choose('10', 20)

// ✅ 正确：显式指定泛型类型参数
let y = choose<string | number>('10', 20)
```

### 场景 3：对象字面量作为参数

```typescript
// ❌ 错误
test.foo('', { id: '', type: 0 })

// ✅ 正确
import { test } from 'test-module'
let option: test.I = { id: '', type: 0 }
test.foo('', option)
```

## 参考

- [华为官方文档 - ArkTS 语法适配规则](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-syntax-adaptation-rules-V5)
- [ArkTS 严格类型检查](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-strict-type-checking-V5)
