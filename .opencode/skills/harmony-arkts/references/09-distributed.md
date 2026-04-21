# HarmonyOS 分布式能力与服务卡片

## 分布式任务调度

HarmonyOS 提供统一的分布式任务调度能力，支持跨设备的应用启动、远程调用和业务迁移。

### 基本约束

- 需要在 Intent 中设置支持分布式的标记
- 需要在 `config.json` 中添加权限申请

```json
"reqPermissions": [
  { "name": "ohos.permission.DISTRIBUTED_DATASYNC" }
]
```

### 分布式启动 Page Ability

```typescript
import featureAbility from '@ohos.ability.featureAbility'

// 获取设备列表
import deviceManager from '@ohos.distributedHardware.deviceManager'

let deviceList = deviceManager.queryDevices(
  deviceManager.DeviceType.DEVICE_TYPE_PHONE
)

// 启动远程 Ability
let want = {
  bundleName: 'com.example.myapp',
  abilityName: 'com.example.myapp.MainAbility',
  deviceId: deviceList[0].deviceId,  // 指定设备
  flags: 0x01000000  // 支持分布式的标记
}

featureAbility.startAbility(want)
```

### 业务迁移

```typescript
import particleAbility from '@ohos.ability.particleAbility'

// 迁移到其他设备
let want = {
  bundleName: 'com.example.myapp',
  abilityName: 'MainAbility',
  deviceId: targetDeviceId
}

particleAbility.missionPool.migrateMission(
  want,
  {
    saveData: true,  // 是否保存数据
    notify: true      // 是否通知
  }
)
```

## 服务卡片

### 概述

服务卡片（Widget）是鸿蒙系统的重要特性，可以将应用的重要信息以卡片形式展示在桌面或其他应用界面中。

### 配置服务卡片

```json
"abilities": [{
  "name": ".MyForm",
  "type": "page",
  "formsEnabled": true,
  "forms": [{
    "name": "MyForm",
    "description": "示例卡片",
    "type": "JS",
    "jsComponentName": "card",
    "colorMode": "auto",
    "isDefault": true,
    "updateEnabled": true,
    "scheduledUpdateTime": "11:00",
    "defaultDimension": "2*2",
    "supportDimensions": ["2*2", "2*4", "4*4"]
  }]
}]
```

### JS 卡片开发

```typescript
// entry/src/main/js/card/pages/index/index.js
import fetch from '@system.fetch'

export default {
  data: {
    title: '卡片标题',
    content: '卡片内容'
  },
  onInit() {
    // 初始化数据
  },
  onUpdate() {
    // 定时更新
  },
  onClick() {
    // 点击事件
  }
}
```
