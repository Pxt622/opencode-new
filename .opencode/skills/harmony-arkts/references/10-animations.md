# ArkTS 动画开发

## 属性动画

```typescript
@State opacity: number = 1

build() {
  Column() {
    Text('Animated')
      .opacity(this.opacity)
      .onClick(() => {
        animateTo({
          duration: 300,
          curve: Curve.EaseOut
        }, () => {
          this.opacity = this.opacity === 1 ? 0 : 1
        })
      })
  }
}
```

## 页面转场

```typescript
// 全局配置
router.pushUrl({
  url: 'pages/Detail'
}, router.RouterMode.Single)

// 自定义转场动画
transition(TransitionEffect.OPACITY.animation({ duration: 300 }))
```
