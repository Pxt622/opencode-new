# HarmonyOS 系统能力 (Syscap) 开发指南

## 什么是系统能力

系统能力（System Capability，简称Syscap）是 HarmonyOS 提供的一套机制，用于检测设备是否支持特定的功能或特性。通过 Syscap 机制，开发者可以：

1. **声明应用所需的最小系统能力**，确保应用只安装在满足条件的设备上
2. **运行时检测设备能力**，根据实际支持情况提供不同功能
3. **设计优雅的降级方案**，在不支持某些功能的设备上提供替代体验

## 系统能力声明

### 在 module.json5 中声明必需的系统能力

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "deviceTypes": [
      "phone",
      "tablet"
    ],
    "systemCapabilities": [
      {
        "name": "SystemCapability.Multimedia.Camera.Core",       // 核心相机能力
        "minApiVersion": 9
      },
      {
        "name": "SystemCapability.ArkUI.ArkUI.Full",            // 完整 ArkUI 能力
        "minApiVersion": 8
      },
      {
        "name": "SystemCapability.Communication.Bluetooth.Core", // 蓝牙核心能力
        "minApiVersion": 7
      }
    ]
  }
}
```

### 系统能力命名规则

系统能力名称遵循分层结构：

```
SystemCapability.<领域>.<子领域>.<具体能力>
```

常见领域：
- **ArkUI**：ArkUI 框架相关能力
- **Multimedia**：多媒体相关能力（相机、音频、视频）
- **Communication**：通信相关能力（蓝牙、NFC、Wi-Fi）
- **Location**：位置服务相关能力
- **DistributedDataManagement**：分布式数据管理
- **Application**：应用框架相关能力
- **Security**：安全相关能力

## 运行时能力检测

### 使用 canIUse API

```typescript
import { canIUse } from '@kit.AbilityKit'

// 检查设备是否支持完整 ArkUI 能力
if (canIUse('SystemCapability.ArkUI.ArkUI.Full')) {
  // 使用完整 ArkUI 功能
  console.log('设备支持完整 ArkUI 能力')
} else {
  // 提供降级 UI
  console.log('设备仅支持基础 ArkUI 能力')
}

// 检查特定 API 是否可用
if (canIUse('SystemCapability.Multimedia.Camera.Camera')) {
  // 相机功能可用
  console.log('设备支持相机功能')
}

// 检查多个能力
const supportedCapabilities = [
  'SystemCapability.ArkUI.ArkUI.Full',
  'SystemCapability.Multimedia.Camera.Core',
  'SystemCapability.Communication.Bluetooth.Core'
]

for (const capability of supportedCapabilities) {
  if (canIUse(capability)) {
    console.log(`设备支持: ${capability}`)
  }
}
```

### 条件编译与动态导入

```typescript
// 动态导入模块（如果支持）
async function loadAdvancedFeatures() {
  try {
    if (canIUse('SystemCapability.Multimedia.Camera.Advanced')) {
      const advancedCamera = await import('@ohos.multimedia.camera.advanced')
      return advancedCamera
    } else {
      const basicCamera = await import('@ohos.multimedia.camera.basic')
      return basicCamera
    }
  } catch (error) {
    console.error('加载相机模块失败:', error)
    return null
  }
}

// 条件渲染组件
@Component
struct AdaptiveComponent {
  @State hasAdvancedFeature: boolean = false
  
  aboutToAppear() {
    this.hasAdvancedFeature = canIUse('SystemCapability.ArkUI.ArkUI.Advanced')
  }
  
  build() {
    Column() {
      if (this.hasAdvancedFeature) {
        AdvancedFeatureComponent()
      } else {
        BasicFeatureComponent()
      }
    }
  }
}
```

## 常见系统能力列表

### ArkUI 相关能力

| 系统能力名称 | 描述 | 最低 API 版本 |
|-------------|------|--------------|
| `SystemCapability.ArkUI.ArkUI.Full` | 完整 ArkUI 框架 | 8 |
| `SystemCapability.ArkUI.ArkUI.Lite` | 轻量级 ArkUI 框架 | 6 |
| `SystemCapability.ArkUI.Component.Advanced` | 高级组件（图表、轮播等） | 9 |
| `SystemCapability.ArkUI.Animation.Advanced` | 高级动画效果 | 9 |

### 多媒体能力

| 系统能力名称 | 描述 | 最低 API 版本 |
|-------------|------|--------------|
| `SystemCapability.Multimedia.Camera.Core` | 基础相机功能 | 7 |
| `SystemCapability.Multimedia.Camera.Advanced` | 高级相机功能（人像模式、夜景等） | 9 |
| `SystemCapability.Multimedia.Audio.Core` | 音频播放/录制 | 7 |
| `SystemCapability.Multimedia.Video.Core` | 视频播放/录制 | 8 |
| `SystemCapability.Multimedia.Image.Core` | 图片处理 | 7 |

### 通信能力

| 系统能力名称 | 描述 | 最低 API 版本 |
|-------------|------|--------------|
| `SystemCapability.Communication.Bluetooth.Core` | 蓝牙基础功能 | 7 |
| `SystemCapability.Communication.Bluetooth.BLE` | 蓝牙低功耗 | 8 |
| `SystemCapability.Communication.WiFi` | Wi-Fi 连接 | 7 |
| `SystemCapability.Communication.NFC.Core` | NFC 基础功能 | 8 |
| `SystemCapability.Communication.NetManager` | 网络管理 | 7 |

### 位置服务

| 系统能力名称 | 描述 | 最低 API 版本 |
|-------------|------|--------------|
| `SystemCapability.Location.Location.Core` | 基础位置服务 | 7 |
| `SystemCapability.Location.Geofence` | 地理围栏 | 8 |
| `SystemCapability.Location.Location.Background` | 后台定位 | 9 |

### 分布式能力

| 系统能力名称 | 描述 | 最低 API 版本 |
|-------------|------|--------------|
| `SystemCapability.DistributedDataManagement.KVStore` | 分布式键值存储 | 8 |
| `SystemCapability.DistributedDataManagement.RelationalStore` | 分布式关系型数据库 | 9 |
| `SystemCapability.DistributedHardware.DeviceManager` | 设备管理 | 8 |
| `SystemCapability.DistributedHardware.DeviceDiscovery` | 设备发现 | 8 |

## 降级方案设计

### 1. 功能降级策略

```typescript
// 相机功能降级示例
class CameraService {
  async takePhoto(): Promise<PhotoResult> {
    if (canIUse('SystemCapability.Multimedia.Camera.Advanced')) {
      // 使用高级相机功能
      return await this.takePhotoWithAdvancedFeatures()
    } else if (canIUse('SystemCapability.Multimedia.Camera.Core')) {
      // 使用基础相机功能
      return await this.takePhotoWithBasicFeatures()
    } else {
      // 不支持相机，使用替代方案
      return await this.useImagePickerFallback()
    }
  }
  
  private async takePhotoWithAdvancedFeatures(): Promise<PhotoResult> {
    // 高级相机功能：HDR、夜景模式、人像模式
    const camera = await import('@ohos.multimedia.camera.advanced')
    return camera.takePhoto({ mode: 'advanced', hdr: true })
  }
  
  private async takePhotoWithBasicFeatures(): Promise<PhotoResult> {
    // 基础相机功能
    const camera = await import('@ohos.multimedia.camera.core')
    return camera.takePhoto({ mode: 'basic' })
  }
  
  private async useImagePickerFallback(): Promise<PhotoResult> {
    // 使用图片选择器作为备选方案
    const picker = await import('@ohos.file.picker')
    return picker.selectImage()
  }
}
```

### 2. UI 降级策略

```typescript
@Component
struct AdaptiveCameraUI {
  @State cameraSupportLevel: 'advanced' | 'basic' | 'none' = 'none'
  
  aboutToAppear() {
    if (canIUse('SystemCapability.Multimedia.Camera.Advanced')) {
      this.cameraSupportLevel = 'advanced'
    } else if (canIUse('SystemCapability.Multimedia.Camera.Core')) {
      this.cameraSupportLevel = 'basic'
    } else {
      this.cameraSupportLevel = 'none'
    }
  }
  
  build() {
    Column() {
      if (this.cameraSupportLevel === 'advanced') {
        // 高级相机界面：多种拍摄模式、专业设置
        AdvancedCameraView()
      } else if (this.cameraSupportLevel === 'basic') {
        // 基础相机界面：简单拍摄功能
        BasicCameraView()
      } else {
        // 无相机支持：显示提示信息和替代方案
        NoCameraView()
      }
    }
  }
}

@Builder
function AdvancedCameraView() {
  Column() {
    Text('专业相机')
      .fontSize(24)
      .fontWeight(FontWeight.Bold)
    
    // 高级功能按钮：HDR、夜景、人像等
    Row() {
      CameraModeButton('HDR')
      CameraModeButton('夜景')
      CameraModeButton('人像')
      CameraModeButton('专业')
    }
    .justifyContent(FlexAlign.SpaceEvenly)
    
    CameraPreview()
  }
}

@Builder
function BasicCameraView() {
  Column() {
    Text('相机')
      .fontSize(20)
    
    // 基础功能：只有拍照按钮
    CameraPreview()
    
    Button('拍照')
      .width(120)
      .height(120)
      .backgroundColor('#FF3A3A')
      .borderRadius(60)
  }
}

@Builder
function NoCameraView() {
  Column() {
    Image($r('app.media.no_camera'))
      .width(120)
      .height(120)
    
    Text('设备不支持相机功能')
      .fontSize(16)
      .fontColor('#666666')
    
    Text('您可以从相册中选择图片')
      .fontSize(14)
      .fontColor('#999999')
      .margin({ top: 8 })
    
    Button('选择图片')
      .margin({ top: 24 })
  }
}
```

### 3. 性能优化建议

1. **减少运行时检测**：在应用启动时检测一次并缓存结果
2. **按需加载模块**：使用动态导入减少初始包大小
3. **预编译分支**：对于已知的设备能力分布，可以预编译不同版本
4. **渐进增强**：先提供基础功能，再按需加载高级功能

```typescript
// 缓存系统能力检测结果
class CapabilityCache {
  private static cache: Map<string, boolean> = new Map()
  
  static check(capability: string): boolean {
    if (!this.cache.has(capability)) {
      const result = canIUse(capability)
      this.cache.set(capability, result)
    }
    return this.cache.get(capability)!
  }
  
  static preloadCommonCapabilities() {
    const commonCapabilities = [
      'SystemCapability.ArkUI.ArkUI.Full',
      'SystemCapability.Multimedia.Camera.Core',
      'SystemCapability.Communication.WiFi',
      'SystemCapability.Location.Location.Core'
    ]
    
    for (const capability of commonCapabilities) {
      this.check(capability)
    }
  }
}

// 应用启动时预加载
onCreate() {
  CapabilityCache.preloadCommonCapabilities()
}
```

## 调试与测试

### 1. 模拟不同设备能力

在 DevEco Studio 中，可以通过以下方式测试不同系统能力：

1. **创建多个设备配置文件**：针对不同能力的设备
2. **使用模拟器能力设置**：在模拟器中配置支持的系统能力
3. **单元测试模拟**：使用测试框架模拟不同能力环境

### 2. 日志记录

```typescript
import { hilog } from '@kit.PerformanceAnalysisKit'

class CapabilityLogger {
  static logCapabilityCheck(capability: string, supported: boolean) {
    hilog.info(
      0x0000,
      'Capability',
      '系统能力检测: %{public}s = %{public}s',
      capability,
      supported ? '支持' : '不支持'
    )
    
    if (!supported) {
      hilog.warn(
        0x0000,
        'Capability',
        '设备不支持能力: %{public}s，将使用降级方案',
        capability
      )
    }
  }
  
  static logFeatureUsage(feature: string, capabilityRequired: string) {
    if (canIUse(capabilityRequired)) {
      hilog.info(
        0x0000,
        'Capability',
        '使用功能: %{public}s (需要: %{public}s)',
        feature,
        capabilityRequired
      )
    }
  }
}
```

### 3. 错误处理

```typescript
function withCapabilityFallback<T>(
  capability: string,
  primaryAction: () => T,
  fallbackAction: () => T
): T {
  try {
    if (canIUse(capability)) {
      CapabilityLogger.logFeatureUsage('primary', capability)
      return primaryAction()
    } else {
      CapabilityLogger.logCapabilityCheck(capability, false)
      return fallbackAction()
    }
  } catch (error) {
    hilog.error(
      0x0000,
      'Capability',
      '能力降级失败: %{public}s, 错误: %{public}s',
      capability,
      JSON.stringify(error)
    )
    return fallbackAction()
  }
}

// 使用示例
const result = withCapabilityFallback(
  'SystemCapability.Multimedia.Camera.Advanced',
  () => advancedCameraFunction(),
  () => basicCameraFunction()
)
```

## 最佳实践

### 1. 明确声明最小能力要求

在 `module.json5` 中准确声明应用所需的最小系统能力，避免应用安装在无法正常运行的设备上。

### 2. 渐进增强设计

采用渐进增强的设计理念：
- 基础功能在所有设备上可用
- 增强功能在支持高级能力的设备上提供
- 优雅降级在不支持某些功能的设备上提供替代方案

### 3. 按需加载资源

```typescript
// 动态加载高级资源
async function loadAdvancedResources() {
  if (!canIUse('SystemCapability.ArkUI.ArkUI.Full')) {
    return // 不需要加载高级资源
  }
  
  // 动态导入高级组件
  const AdvancedChart = await import('./advanced/AdvancedChart.ets')
  const AdvancedMap = await import('./advanced/AdvancedMap.ets')
  
  return { AdvancedChart, AdvancedMap }
}
```

### 4. 测试覆盖

确保测试覆盖：
- 所有声明的系统能力
- 所有降级路径
- 不同能力组合下的用户体验

## 参考

- [华为官方文档 - 系统能力开发指导](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/syscap-development)
- [系统能力列表参考](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/syscap-list)
- [canIUse API 文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-abilitykit-caniuse)

> 注意：系统能力名称和可用性可能随 HarmonyOS 版本更新而变化，建议定期查阅最新官方文档。