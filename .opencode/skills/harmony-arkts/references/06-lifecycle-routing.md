# ArkTS 生命周期与路由导航

## 组件生命周期

| 状态 | 说明 |
|------|------|
| `aboutToAppear` | 组件即将显示 |
| `aboutToDisappear` | 组件即将销毁 |
| `onPageShow` | 页面显示（@Entry 组件） |
| `onPageHide` | 页面隐藏（@Entry 组件） |

## Ability 生命周期

| 状态 | 说明 |
|------|------|
| `onCreate` | Ability 创建 |
| `onDestroy` | Ability 销毁 |
| `onForeground` | 应用进入前台 |
| `onBackground` | 应用进入后台 |
| `onWindowStageCreate` | 窗口创建 |
| `onWindowStageDestroy` | 窗口销毁 |

### EntryAbility.ets 标准模板

```typescript
import { AbilityConstant, UIAbility, Want } from '@kit.AbilityKit'
import { hilog } from '@kit.PerformanceAnalysisKit'
import { window } from '@kit.ArkUI'

export default class EntryAbility extends UIAbility {
  onCreate(want: Want, launchParam: AbilityConstant.LaunchParam): void {
    hilog.info(0x0000, 'TestTag', '%{public}s', 'Ability onCreate')
  }

  onDestroy(): void {
    hilog.info(0x0000, 'TestTag', '%{public}s', 'Ability onDestroy')
  }

  onWindowStageCreate(windowStage: window.WindowStage): void {
    hilog.info(0x0000, 'TestTag', '%{public}s', 'Ability onWindowStageCreate')

    // ✅ 正确：直接加载首页内容
    windowStage.loadContent('pages/Index', (err, data) => {
      if (err.code) {
        hilog.error(0x0000, 'TestTag', 'Failed to load the content. Cause: %{public}s', JSON.stringify(err))
        return
      }
      hilog.info(0x0000, 'TestTag', 'Succeeded in loading the content.')
    })
  }

  onWindowStageDestroy(): void {
    hilog.info(0x0000, 'TestTag', '%{public}s', 'Ability onWindowStageDestroy')
  }

  onForeground(): void {
    hilog.info(0x0000, 'TestTag', '%{public}s', 'Ability onForeground')
  }

  onBackground(): void {
    hilog.info(0x0000, 'TestTag', '%{public}s', 'Ability onBackground')
  }
}
```

> ⚠️ **重要警告**：在 `onWindowStageCreate` 中不要调用 `windowStage.getMainWindow().setWindowBackgroundColor()`，因为窗口对象可能尚未完全初始化，会导致应用启动崩溃（Error code: 1300002）。如果需要设置背景色，请在页面组件的 `.backgroundColor()` 中设置。

### ❌ 错误示例（导致启动崩溃）

```typescript
onWindowStageCreate(windowStage: window.WindowStage): void {
  // ❌ 错误：窗口未初始化完成，调用会失败
  windowStage.getMainWindow((err, data) => {
    if (err.code) {
      return
    }
    data.setWindowBackgroundColor('#F5F5F5')  // 抛出异常，导致应用崩溃
  })
  
  windowStage.loadContent('pages/Index', ...)
}
```

### ✅ 正确示例

```typescript
onWindowStageCreate(windowStage: window.WindowStage): void {
  // ✅ 正确：直接加载内容，不在此处设置窗口属性
  windowStage.loadContent('pages/Index', (err, data) => {
    if (err.code) {
      hilog.error(0x0000, 'TestTag', 'Failed to load the content. Cause: %{public}s', JSON.stringify(err))
      return
    }
    hilog.info(0x0000, 'TestTag', 'Succeeded in loading the content.')
  })
}

// 在页面组件中设置背景色
@Entry
@Component
struct Index {
  build() {
    Column() {
      // 页面内容
    }
    .width('100%')
    .height('100%')
    .backgroundColor('#F5F5F5')  // ✅ 在这里设置背景色
  }
}
```

## Service Ability 生命周期

| 状态 | 说明 |
|------|------|
| `onStart` | Service 启动 |
| `onCommand` | 接收命令 |
| `onConnect` | 连接成功 |
| `onDisconnect` | 断开连接 |
| `onStop` | Service 停止 |

## ArkTS 路由

```typescript
import router from '@ohos.router'

// 跳转到指定页面
router.pushUrl({
  url: 'pages/Detail',
  params: {
    id: 1,
    name: 'Test'
  }
})

// 获取参数
onPageShow() {
  const params = router.getParams()
  console.log('id:', params.id)
}

// 返回上一页
router.back()

// 返回到指定页面
router.back({ url: 'pages/Index' })
```

## 页面路由配置

在 `resources/base/profile/main_pages.json` 中声明页面：

```json
{
  "src": [
    "pages/Index",
    "pages/Detail"
  ]
}
```

## 窗口操作注意事项

### 何时可以安全操作窗口

| 时机 | 是否安全 | 说明 |
|------|---------|------|
| `onWindowStageCreate` | ⚠️ 部分操作 | 可以调用 `loadContent`，但不建议操作窗口属性 |
| 页面 `onPageShow` | ✅ 安全 | 页面已显示，可以操作窗口 |
| 页面 `aboutToAppear` | ⚠️ 谨慎 | 组件即将显示，部分窗口操作可能失败 |
| 按钮点击回调 | ✅ 安全 | 用户交互时窗口已完全初始化 |

### 设置窗口属性的正确方式

```typescript
// 在页面显示后设置窗口属性
@Entry
@Component
struct Index {
  onPageShow() {
    // ✅ 获取窗口并设置属性
    let windowClass = window.getLastWindow(getContext(this))
    windowClass.then((data) => {
      // 设置沉浸式状态栏
      data.setWindowLayoutFullScreen(true)
      // 设置状态栏文字颜色
      data.setWindowSystemBarProperties({
        statusBarContentColor: '#FFFFFF'
      })
    })
  }
}
```

## 参考

- [华为官方文档 - UIAbility 生命周期](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/uiability-lifecycle)
- [窗口开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/window-development-guide)
