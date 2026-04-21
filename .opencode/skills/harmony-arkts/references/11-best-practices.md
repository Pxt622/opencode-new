# HarmonyOS ArkTS 最佳实践

## 1. 组件设计原则

- **单一职责**：一个组件只做一件事
- **复用性**：通过 @Builder 抽取复用 UI
- **解耦**：业务逻辑与 UI 分离
- **类型安全**：所有对象字面量必须有显式类型声明

## 2. 性能优化

- 减少不必要的状态更新
- 使用懒加载 (LazyForEach)
- 避免在 build() 中创建新对象
- 使用 `recycle` 复用资源

## 3. 状态管理准则

- 最小化状态：只将需要 UI 更新的数据设为状态
- 单向数据流：优先使用 @Prop 而不是 @Link
- 预定义常量：避免在 @State 中直接使用对象字面量

## 4. 布局优化

- 减少嵌套层级
- 优先使用 Column/Row 而非 Flex（简单场景）
- 固定高度/宽度避免性能消耗

## 5. 类型安全最佳实践

### 5.1 接口定义规范

```typescript
// ✅ 正确：所有嵌套对象都提取为独立接口
interface Coordinates {
  latitude: number
  longitude: number
}

interface Waypoint {
  id: number
  name: string
  type: 'start' | 'end' | 'waypoint'
  coordinates: Coordinates  // 使用独立接口，而非内联对象
}

interface RoutePlan {
  id: number
  startPoint: string
  endPoint: string
  transitTime: string
  distance: string
  waypoints: Waypoint[]
}
```

### 5.2 组件状态初始化

```typescript
// ❌ 错误：在 @State 中直接使用对象字面量数组
@Component
struct MapComponent {
  @State routePlan: RoutePlan = {
    waypoints: [
      { id: 1, name: 'Paris', coordinates: { latitude: 48.8, longitude: 2.3 } },
      { id: 2, name: 'NYC', coordinates: { latitude: 40.7, longitude: -74.0 } }
    ]
  }
}

// ✅ 正确：使用预定义的私有成员变量
@Component
struct MapComponent {
  private coord1: Coordinates = { latitude: 48.8, longitude: 2.3 }
  private coord2: Coordinates = { latitude: 40.7, longitude: -74.0 }
  
  private wp1: Waypoint = { id: 1, name: 'Paris', coordinates: this.coord1 }
  private wp2: Waypoint = { id: 2, name: 'NYC', coordinates: this.coord2 }
  
  @State routePlan: RoutePlan = {
    waypoints: [this.wp1, this.wp2]
  }
}
```

### 5.3 常量数据定义

```typescript
// ✅ 正确：在模块级别预定义常量
const waypoint1: Waypoint = {
  id: 1,
  name: 'Paris, France',
  type: 'start',
  coordinates: { latitude: 48.8566, longitude: 2.3522 }
}

const waypoint2: Waypoint = {
  id: 2,
  name: 'New York, USA',
  type: 'waypoint',
  coordinates: { latitude: 40.7128, longitude: -74.0060 }
}

export const MOCK_ROUTE_PLAN: RoutePlan = {
  id: 1,
  startPoint: 'Paris, France',
  endPoint: 'Arizona, USA',
  transitTime: '15小时',
  distance: '8,500公里',
  waypoints: [waypoint1, waypoint2]
}
```

## 6. 窗口操作安全指南

### 6.1 启动阶段避免操作窗口

```typescript
// ❌ 错误：在 onWindowStageCreate 中操作窗口属性
onWindowStageCreate(windowStage: window.WindowStage): void {
  windowStage.getMainWindow((err, data) => {
    data.setWindowBackgroundColor('#F5F5F5')  // 可能导致崩溃
  })
}

// ✅ 正确：直接加载内容，不操作窗口
onWindowStageCreate(windowStage: window.WindowStage): void {
  windowStage.loadContent('pages/Index', (err) => {
    if (err.code) {
      hilog.error(0x0000, 'Tag', 'Failed: %{public}s', JSON.stringify(err))
    }
  })
}
```

### 6.2 在页面中设置背景色

```typescript
@Entry
@Component
struct Index {
  build() {
    Column() {
      // 页面内容
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')  // ✅ 在组件中设置背景色
  }
}
```

## 7. 项目配置检查清单

### 7.1 首次创建项目后必须检查

- [ ] `AppScope/app.json5` 存在且格式正确
- [ ] `entry/src/main/module.json5` 中 `startWindowIcon` 已配置
- [ ] `entry/src/main/module.json5` 中 `icon` 已配置
- [ ] `build-profile.json5` 中 SDK 版本号为字符串格式（如 `"5.0.0(12)"`）
- [ ] `AppScope/app.json5` 中 `apiReleaseType` 格式正确（如 `"Release1"`）
- [ ] `user_grant` 权限包含 `reason` 和 `usedScene`
- [ ] `resources/base/media/` 目录包含图标文件
- [ ] `resources/base/profile/main_pages.json` 页面路由正确

### 7.2 构建前检查

- [ ] 所有对象字面量都有显式类型声明
- [ ] 数组字面量元素类型可推断
- [ ] 接口中没有内联对象字面量类型
- [ ] @Builder 方法不支持链式调用 `.position()` 等属性

## 8. 调试技巧

### 8.1 查看设备日志

```bash
# 使用 hdc 查看日志
hdc hilog | grep InstantTrip

# 在 DevEco Studio 中查看 Log 窗口
# 过滤条件：tag:InstantTrip
```

### 8.2 常见崩溃原因

| 现象 | 可能原因 | 解决方案 |
|------|---------|---------|
| 白屏后闪退 | `setWindowBackgroundColor` 调用失败 | 移除窗口背景色设置，在页面组件中设置 |
| 构建失败 | `startWindowIcon` 缺失 | 在 `module.json5` 中添加 |
| 构建失败 | 权限 `reason` 缺失 | 为 `user_grant` 权限添加 `reason` 和 `usedScene` |
| 编译错误 | 对象字面量无类型 | 提取为独立接口或添加类型注解 |
| 编译错误 | 数组元素类型不明确 | 为数组变量添加类型注解 |

## 9. 设计令牌 (Design Tokens) 使用建议

```typescript
export class DesignTokens {
  // 主色调
  static readonly PRIMARY: string = '#1A1A1A'
  static readonly ACCENT: string = '#E6FF00'
  static readonly BACKGROUND: string = '#F5F5F5'
  static readonly CARD_BACKGROUND: string = '#FFFFFF'

  // 文字颜色
  static readonly TEXT_PRIMARY: string = '#1A1A1A'
  static readonly TEXT_SECONDARY: string = '#666666'
  static readonly TEXT_HINT: string = '#999999'

  // 圆角
  static readonly RADIUS_SMALL: number = 8
  static readonly RADIUS_MEDIUM: number = 16
  static readonly RADIUS_LARGE: number = 24

  // 字体大小
  static readonly FONT_SIZE_XS: number = 12
  static readonly FONT_SIZE_SM: number = 14
  static readonly FONT_SIZE_MD: number = 16
  static readonly FONT_SIZE_LG: number = 20
  static readonly FONT_SIZE_XL: number = 24
}
```

## 参考

- [华为官方文档 - ArkTS 最佳实践](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-best-practices)
- [性能优化指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/performance-optimization)
