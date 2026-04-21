# ArkTS 状态管理

## 装饰器对比

| 装饰器 | 作用域 | 说明 |
|--------|--------|------|
| `@State` | 组件内 | 组件私有状态，变化触发 UI 更新 |
| `@Prop` | 父→子 | 单向数据传递 |
| `@Link` | 父↔子 | 双向数据绑定 |
| `@Provide` | 祖先→后代 | 跨层级数据提供 |
| `@Consume` | 后代←祖先 | 消费 Provide 数据 |
| `@Observed` | 类级别 | 配合 @ObjectLink 监听对象变化 |
| `@ObjectLink` | 组件内 | 监听 @Observed 对象的变化 |

## 简单示例

```typescript
@Entry
@Component
struct Counter {
  @State count: number = 0

  build() {
    Column() {
      Text(`Count: ${this.count}`)
        .fontSize(30)

      Row() {
        Button('-')
          .onClick(() => this.count--)

        Button('+')
          .onClick(() => this.count++)
      }
    }
  }
}
```

## 父子双向绑定

```typescript
// 父组件
@Entry
@Component
struct Parent {
  @State fatherMsg: string = 'Hello'

  build() {
    Column() {
      Child({
        message: $fatherMsg  // 使用 $ 创建双向绑定
      })
    }
  }
}

// 子组件
@Component
struct Child {
  @Link message: string  // @Link 实现双向绑定

  build() {
    Column() {
      Text(this.message)
      TextInput({ text: this.message })
        .onChange((value: string) => {
          this.message = value
        })
    }
  }
}
```
