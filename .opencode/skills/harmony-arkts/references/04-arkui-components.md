# ArkUI 声明式 UI 与布局组件

## 基础组件示例

```typescript
@Entry
@Component
struct Index {
  @State message: string = 'Hello ArkTS'

  build() {
    Column() {
      Text(this.message)
        .fontSize(50)
        .fontWeight(FontWeight.Bold)

      Button('Click Me')
        .onClick(() => {
          this.message = 'Button Clicked!'
        })
    }
    .width('100%')
    .height('100%')
    .justifyContent(FlexAlign.Center)
  }
}
```

## 常用布局容器

| 容器 | 说明 | 关键属性 |
|------|------|---------|
| `Column` | 垂直布局 | `justifyContent`, `alignItems` |
| `Row` | 水平布局 | `justifyContent`, `alignItems` |
| `Stack` | 层叠布局 | `alignContent` |
| `Flex` | 弹性布局 | `direction`, `wrap` |
| `Grid` | 网格布局 | `columnsTemplate`, `rowsTemplate` |
| `List` | 列表布局 | 配合 `ListItem` 使用 |
| `Swiper` | 轮播组件 | `autoPlay`, `indicator` |
| `Tabs` | 标签页 | 配合 `TabBar` 使用 |

## 常用组件

| 组件 | 说明 | 关键属性/方法 |
|------|------|---------------|
| `Text` | 文本 | `fontSize`, `fontColor`, `fontWeight` |
| `Image` | 图片 | `objectFit`, `alt` |
| `Button` | 按钮 | `type`, `onClick` |
| `TextInput` | 文本输入 | `placeholder`, `onChange` |
| `Checkbox` | 复选框 | `checked`, `onChange` |
| `Radio` | 单选按钮 | `checked`, `group` |
| `Switch` | 开关 | `isOn`, `onChange` |
| `Slider` | 滑块 | `value`, `min`, `max` |
| `Progress` | 进度条 | `value`, `type` |
| `LoadingProgress` | 加载中 | `loadingDuration` |

## 常见布局模式

### 页面布局分解

```
页面
├── 标题区域
│   ├── 图标
│   └── 标题文字
├── 内容区域
│   ├── 列表项
│   └── 列表项
└── 底部操作区
```

### Flex 布局技巧

```typescript
// 水平居中垂直居中
Row() {
  Text('Center')
}
.width('100%')
.height('100%')
.justifyContent(FlexAlign.Center)
.alignItems(VerticalAlign.Center)

// 均匀分布
Row() {
  Button('A')
  Button('B')
  Button('C')
}
.width('100%')
.justifyContent(FlexAlign.SpaceEvenly)
```

### 列表布局

```typescript
@State dataSource: MyDataSource = new MyDataSource()

build() {
  List() {
    LazyForEach(this.dataSource, (item: Item) => {
      ListItem() {
        ListItemComponent({ item: item })
      }
    }, (item: Item) => item.id.toString())
  }
  .width('100%')
  .height('100%')
}
```
