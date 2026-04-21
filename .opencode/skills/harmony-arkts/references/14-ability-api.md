# HarmonyOS Ability API 开发指南

## Ability 概述

Ability 是 HarmonyOS 应用的重要组成部分，是应用所具备能力的抽象。一个应用可以包含多个 Ability，每个 Ability 都可以被独立地启动和使用。

### Ability 类型

| 类型 | 模板 | 描述 | 特点 |
|------|------|------|------|
| **FA (Feature Ability)** | Page 模板 | 有 UI 界面，提供与用户交互的能力 | 支持页面路由、生命周期管理、状态保存 |
| **PA (Particle Ability)** | Service 模板 | 无 UI 界面，提供后台运行任务的能力 | 支持后台服务、跨进程通信、长期运行 |
| **PA (Particle Ability)** | Data 模板 | 无 UI 界面，提供数据访问抽象的能力 | 支持统一数据访问、数据共享、数据安全 |

### Ability 与组件的关系

```
应用 (Application)
├── FA (Page Ability) - 页面 Ability，有 UI
│   ├── Page - 页面，对应一个 UI 界面
│   ├── Page - 另一个页面
│   └── ...
├── PA (Service Ability) - 服务 Ability，无 UI
└── PA (Data Ability) - 数据 Ability，无 UI
```

## FA (Feature Ability) 开发

### Page Ability 基本结构

```typescript
import UIAbility from '@ohos.app.ability.UIAbility'
import window from '@ohos.window'

export default class EntryAbility extends UIAbility {
  // Ability 创建时调用
  onCreate(want, launchParam) {
    console.log('EntryAbility onCreate')
  }

  // 窗口阶段创建时调用
  onWindowStageCreate(windowStage: window.WindowStage) {
    console.log('EntryAbility onWindowStageCreate')
    
    // 加载主页面
    windowStage.loadContent('pages/Index', (err) => {
      if (err.code) {
        console.error(`Failed to load content. Code: ${err.code}, Message: ${err.message}`)
        return
      }
      console.log('Succeeded in loading content')
    })
  }

  // 窗口阶段销毁时调用
  onWindowStageDestroy() {
    console.log('EntryAbility onWindowStageDestroy')
  }

  // Ability 进入前台时调用
  onForeground() {
    console.log('EntryAbility onForeground')
  }

  // Ability 进入后台时调用
  onBackground() {
    console.log('EntryAbility onBackground')
  }

  // Ability 销毁时调用
  onDestroy() {
    console.log('EntryAbility onDestroy')
  }
}
```

### Page Ability 配置

在 `module.json5` 中配置 Page Ability：

```json5
{
  "module": {
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "description": "$string:EntryAbility_desc",
        "icon": "$media:icon",
        "label": "$string:EntryAbility_label",
        "startWindowIcon": "$media:icon",
        "startWindowBackground": "$color:start_window_background",
        "exported": true,
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]
          }
        ],
        "backgroundModes": ["dataTransfer", "location"], // 后台模式
        "orientation": "unspecified", // 屏幕方向
        "supportWindowMode": ["fullscreen", "split", "floating"], // 窗口模式支持
        "window": {
          "designWidth": 720,
          "autoDesignWidth": false
        }
      }
    ]
  }
}
```

### Page Ability 生命周期

```
onCreate → onWindowStageCreate → onForeground
        ↓
   onBackground
        ↓
   onForeground
        ↓
onWindowStageDestroy → onDestroy
```

**详细生命周期说明**：

| 生命周期方法 | 触发时机 | 典型操作 |
|-------------|---------|---------|
| `onCreate` | Ability 实例创建时 | 初始化全局数据、注册监听器 |
| `onWindowStageCreate` | 窗口阶段创建时 | 加载页面内容、设置窗口属性 |
| `onForeground` | Ability 切换到前台时 | 恢复数据、开始动画、连接服务 |
| `onBackground` | Ability 切换到后台时 | 保存数据、暂停动画、释放资源 |
| `onWindowStageDestroy` | 窗口阶段销毁时 | 清理窗口相关资源 |
| `onDestroy` | Ability 销毁时 | 释放所有资源、取消监听 |

### 页面路由与导航

```typescript
import router from '@ohos.router'

// 跳转到指定页面
router.pushUrl({
  url: 'pages/Detail',
  params: { id: 123, name: '商品详情' }
})

// 带转场动画的跳转
router.pushUrl({
  url: 'pages/Detail'
}, router.RouterMode.Single, (err) => {
  if (err) {
    console.error(`Push url failed. Code: ${err.code}, message: ${err.message}`)
    return
  }
  console.log('Push url succeeded')
})

// 返回上一页
router.back()

// 返回指定页面
router.back({
  url: 'pages/Home'
})

// 替换当前页面
router.replaceUrl({
  url: 'pages/NewPage'
})

// 清空页面栈并跳转到新页面
router.clear()
router.pushUrl({
  url: 'pages/Login'
})

// 获取页面参数
const params = router.getParams()
console.log('页面参数:', params)
```

## PA (Particle Ability) 开发

### Service Ability

Service Ability 用于在后台执行任务或提供长期运行的服务。

```typescript
import ServiceExtensionAbility from '@ohos.app.ability.ServiceExtensionAbility'
import rpc from '@ohos.rpc'

export default class MyServiceAbility extends ServiceExtensionAbility {
  // Service 创建时调用
  onCreate(want) {
    console.log('MyServiceAbility onCreate')
  }

  // Service 销毁时调用
  onDestroy() {
    console.log('MyServiceAbility onDestroy')
  }

  // 连接建立时调用
  onConnect(want) {
    console.log('MyServiceAbility onConnect')
    
    // 返回 IRemoteObject 对象，用于进程间通信
    return new MyRemoteObject('MyService')
  }

  // 连接断开时调用
  onDisconnect(want) {
    console.log('MyServiceAbility onDisconnect')
  }

  // 命令请求时调用
  onCommand(want, startId) {
    console.log('MyServiceAbility onCommand')
  }
}

// 远程对象实现
class MyRemoteObject extends rpc.RemoteObject {
  constructor(descriptor) {
    super(descriptor)
  }

  // 处理远程调用
  onRemoteRequest(code, data, reply, option) {
    console.log(`onRemoteRequest code: ${code}`)
    
    switch (code) {
      case 1: // 自定义命令1
        const name = data.readString()
        console.log(`Received name: ${name}`)
        reply.writeString(`Hello, ${name}!`)
        break
      case 2: // 自定义命令2
        const value = data.readInt()
        console.log(`Received value: ${value}`)
        reply.writeInt(value * 2)
        break
      default:
        console.log(`Unknown request code: ${code}`)
        break
    }
    
    return true
  }
}
```

### Service Ability 配置

```json5
{
  "module": {
    "abilities": [
      {
        "name": "MyServiceAbility",
        "type": "service",
        "srcEntry": "./ets/myserviceability/MyServiceAbility.ets",
        "description": "$string:MyServiceAbility_desc",
        "icon": "$media:icon",
        "label": "$string:MyServiceAbility_label",
        "exported": true, // 是否允许其他应用访问
        "backgroundModes": ["dataTransfer"], // 后台模式
        "visible": true // 是否可见（可被其他应用发现）
      }
    ]
  }
}
```

### Data Ability

Data Ability 提供统一的数据访问接口，支持跨应用数据共享。

```typescript
import dataAbility from '@ohos.data.dataAbility'
import dataRdb from '@ohos.data.rdb'
import valuesBucket from '@ohos.data.valuesBucket'

const TABLE_NAME = 'user'
const STORE_CONFIG = { name: 'UserStore.db' }

export default class UserDataAbility extends dataAbility.DataAbility {
  private rdbStore: dataRdb.RdbStore | null = null

  // Data Ability 创建时调用
  onCreate(want, callback) {
    console.log('UserDataAbility onCreate')
    
    // 创建数据库
    const context = this.context
    dataRdb.getRdbStore(context, STORE_CONFIG, 1, (err, store) => {
      if (err) {
        console.error(`Failed to get RdbStore. Code: ${err.code}, message: ${err.message}`)
        callback()
        return
      }
      
      this.rdbStore = store
      
      // 创建表
      const sql = `CREATE TABLE IF NOT EXISTS ${TABLE_NAME} (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        age INTEGER,
        email TEXT
      )`
      
      store.executeSql(sql, [], (err) => {
        if (err) {
          console.error(`Failed to create table. Code: ${err.code}, message: ${err.message}`)
        }
        callback()
      })
    })
  }

  // 插入数据
  insert(uri, valueBucket, callback) {
    console.log('UserDataAbility insert')
    
    if (!this.rdbStore) {
      callback(-1)
      return
    }
    
    this.rdbStore.insert(TABLE_NAME, valueBucket, callback)
  }

  // 查询数据
  query(uri, columns, predicates, callback) {
    console.log('UserDataAbility query')
    
    if (!this.rdbStore) {
      callback(null, [])
      return
    }
    
    this.rdbStore.query(TABLE_NAME, predicates, columns, callback)
  }

  // 更新数据
  update(uri, valueBucket, predicates, callback) {
    console.log('UserDataAbility update')
    
    if (!this.rdbStore) {
      callback(0)
      return
    }
    
    this.rdbStore.update(valueBucket, predicates, callback)
  }

  // 删除数据
  delete(uri, predicates, callback) {
    console.log('UserDataAbility delete')
    
    if (!this.rdbStore) {
      callback(0)
      return
    }
    
    this.rdbStore.delete(predicates, callback)
  }
}
```

### Data Ability 配置

```json5
{
  "module": {
    "abilities": [
      {
        "name": "UserDataAbility",
        "type": "data",
        "srcEntry": "./ets/userdataability/UserDataAbility.ets",
        "uri": "dataability://com.example.myapp.UserDataAbility",
        "visible": true,
        "permissions": [
          "ohos.permission.READ_USER_DATA",
          "ohos.permission.WRITE_USER_DATA"
        ]
      }
    ]
  }
}
```

## Ability 间通信

### 启动 Ability

```typescript
import UIAbility from '@ohos.app.ability.UIAbility'
import wantManager from '@ohos.app.ability.wantManager'

// 启动同一应用内的 Ability
let want = {
  bundleName: 'com.example.myapp',
  abilityName: 'com.example.myapp.DetailAbility',
  parameters: { // 传递参数
    id: 123,
    title: '商品详情'
  }
}

// 启动其他应用的 Ability
let externalWant = {
  bundleName: 'com.example.otherapp',
  abilityName: 'com.example.otherapp.MainAbility',
  action: 'action.view.details',
  entities: ['entity.system.browser'],
  uri: 'https://example.com/data',
  flags: 0x10000000 // 标志位
}

// 显式启动
this.context.startAbility(want, (err) => {
  if (err.code) {
    console.error(`Failed to start ability. Code: ${err.code}, message: ${err.message}`)
    return
  }
  console.log('Succeeded in starting ability')
})

// 隐式启动（根据 action 和 entities 匹配）
this.context.startAbility(externalWant, (err) => {
  if (err.code) {
    console.error(`Failed to start ability. Code: ${err.code}, message: ${err.message}`)
    return
  }
  console.log('Succeeded in starting ability')
})
```

### 连接 Service Ability

```typescript
import featureAbility from '@ohos.ability.featureAbility'

// 连接 Service Ability
let want = {
  bundleName: 'com.example.myapp',
  abilityName: 'com.example.myapp.MyServiceAbility'
}

let connection = {
  onConnect: (elementName, proxy) => {
    console.log('Service connected')
    
    // 发送消息到 Service
    let data = rpc.MessageParcel.create()
    data.writeString('Hello Service')
    let reply = rpc.MessageParcel.create()
    
    proxy.sendRequest(1, data, reply, { async: false })
      .then(() => {
        const result = reply.readString()
        console.log(`Service reply: ${result}`)
      })
      .catch((err) => {
        console.error(`Send request failed: ${err.message}`)
      })
  },
  onDisconnect: (elementName) => {
    console.log('Service disconnected')
  },
  onFailed: (code) => {
    console.error(`Connect service failed. Code: ${code}`)
  }
}

// 建立连接
let connectId = featureAbility.connectAbility(want, connection)

// 断开连接
featureAbility.disconnectAbility(connectId)
```

### 访问 Data Ability

```typescript
import dataAbility from '@ohos.data.dataAbility'
import ohos_data_ability from '@ohos.data.dataAbility'

// 创建 Data Ability Helper
let helper = dataAbility.createDataAbilityHelper('dataability://com.example.myapp.UserDataAbility')

// 插入数据
let values = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
}

helper.insert('', values, (err, data) => {
  if (err) {
    console.error(`Insert failed. Code: ${err.code}, message: ${err.message}`)
    return
  }
  console.log(`Insert succeeded. ID: ${data}`)
})

// 查询数据
let predicates = new dataAbility.DataAbilityPredicates()
predicates.equalTo('name', '张三')

helper.query('', ['id', 'name', 'age', 'email'], predicates, (err, data) => {
  if (err) {
    console.error(`Query failed. Code: ${err.code}, message: ${err.message}`)
    return
  }
  console.log(`Query result: ${JSON.stringify(data)}`)
})

// 使用 URI 访问
let uri = 'dataability://com.example.myapp.UserDataAbility/user/1'
let uriHelper = ohos_data_ability.createDataAbilityHelper(uri)
```

## Ability 配置详解

### skills 配置

skills 定义 Ability 能够响应的意图（Intent）：

```json5
"skills": [
  {
    "entities": ["entity.system.home"], // 实体类型
    "actions": ["action.system.home"],   // 动作类型
    "uris": [
      {
        "scheme": "https",
        "host": "example.com",
        "port": "443",
        "path": "/*",
        "type": "text/*"
      }
    ]
  },
  {
    "entities": ["entity.system.browser"],
    "actions": ["action.system.view"],
    "uris": [
      {
        "scheme": "http",
        "host": "*",
        "port": "*",
        "path": "/*",
        "type": "text/html"
      }
    ]
  }
]
```

### 启动模式配置

```json5
"launchType": "standard" // 或 "singleton"
```

- **standard**（标准模式）：每次启动都会创建新的 Ability 实例
- **singleton**（单例模式）：整个系统中只有一个该 Ability 的实例

### 多实例配置

```json5
"multipleProcesses": true, // 是否支持多进程
"process": ":remote",      // 进程名称
"supportMultiUser": true   // 是否支持多用户
```

## 高级特性

### Ability 迁移

Ability 迁移允许用户将正在运行的 Ability 从一个设备迁移到另一个设备。

```typescript
import UIAbility from '@ohos.app.ability.UIAbility'
import distributedMissionManager from '@ohos.distributedMissionManager'

export default class MigratableAbility extends UIAbility {
  private missionId: number = -1

  onCreate(want, launchParam) {
    // 注册迁移回调
    this.context.registerAbilityMigrationCallback({
      // 保存迁移数据
      onSaveData: (saveData) => {
        console.log('onSaveData called')
        saveData['customData'] = {
          currentPage: 'pages/Detail',
          selectedItem: 123,
          userProgress: 75
        }
        return true
      },
      
      // 恢复迁移数据
      onRestoreData: (restoreData) => {
        console.log('onRestoreData called')
        const customData = restoreData['customData']
        if (customData) {
          console.log(`Restored data: ${JSON.stringify(customData)}`)
          // 根据恢复的数据恢复 UI 状态
        }
        return true
      }
    })
  }

  // 开始迁移
  startMigration(targetDeviceId: string) {
    let want = {
      deviceId: targetDeviceId,
      bundleName: this.context.abilityInfo.bundleName,
      abilityName: this.context.abilityInfo.name
    }

    distributedMissionManager.startSyncRemoteMissions({
      deviceIds: [targetDeviceId]
    }, (err) => {
      if (err) {
        console.error(`Start sync missions failed: ${err.message}`)
        return
      }
      
      distributedMissionManager.continueMission(want, this.missionId, {
        deviceId: targetDeviceId
      }, (err) => {
        if (err) {
          console.error(`Continue mission failed: ${err.message}`)
          return
        }
        console.log('Mission migration started')
      })
    })
  }
}
```

### Ability 卡片

Ability 卡片是 Ability 的轻量化展示形式。

```typescript
import FormExtensionAbility from '@ohos.app.form.FormExtensionAbility'
import formBindingData from '@ohos.app.form.formBindingData'

export default class MyFormAbility extends FormExtensionAbility {
  onAddForm(want) {
    console.log('MyFormAbility onAddForm')
    
    // 创建卡片数据
    let formData = {
      title: '我的卡片',
      content: '卡片内容',
      updateTime: new Date().toLocaleTimeString()
    }
    
    return formBindingData.createFormBindingData(formData)
  }

  onCastToNormalForm(formId) {
    console.log(`MyFormAbility onCastToNormalForm, formId: ${formId}`)
  }

  onUpdateForm(formId) {
    console.log(`MyFormAbility onUpdateForm, formId: ${formId}`)
    
    // 更新卡片数据
    let newData = {
      title: '更新的卡片',
      content: '已更新内容',
      updateTime: new Date().toLocaleTimeString()
    }
    
    return formBindingData.createFormBindingData(newData)
  }

  onChangeFormVisibility(newStatus) {
    console.log(`MyFormAbility onChangeFormVisibility, newStatus: ${newStatus}`)
  }

  onFormEvent(formId, message) {
    console.log(`MyFormAbility onFormEvent, formId: ${formId}, message: ${message}`)
  }

  onRemoveForm(formId) {
    console.log(`MyFormAbility onRemoveForm, formId: ${formId}`)
  }

  onAcquireFormState(want) {
    console.log('MyFormAbility onAcquireFormState')
    return formInfo.FormState.READY
  }
}
```

## 错误处理与调试

### 常见错误码

| 错误码 | 描述 | 解决方案 |
|--------|------|---------|
| 1600001 | Ability 不存在 | 检查 bundleName 和 abilityName 是否正确 |
| 1600002 | Ability 正在拉起中 | 等待当前启动完成后再尝试 |
| 1600003 | 连接 Ability 失败 | 检查目标 Ability 是否已启动，网络连接是否正常 |
| 1600004 | 启动 Ability 失败 | 检查权限声明和 exported 属性 |
| 1600006 | 连接 Service Ability 失败 | 检查 Service Ability 配置和状态 |

### 调试技巧

```typescript
// 1. 记录 Ability 生命周期
import { hilog } from '@kit.PerformanceAnalysisKit'

export default class DebuggableAbility extends UIAbility {
  onCreate(want, launchParam) {
    hilog.info(0x0000, 'Ability', 'onCreate: want=%{public}s', JSON.stringify(want))
  }
  
  onWindowStageCreate(windowStage) {
    hilog.info(0x0000, 'Ability', 'onWindowStageCreate')
    
    // 2. 检查窗口状态
    windowStage.getMainWindow((err, window) => {
      if (err) {
        hilog.error(0x0000, 'Ability', 'getMainWindow failed: %{public}d', err.code)
        return
      }
      hilog.info(0x0000, 'Ability', 'Window ready: %{public}s', window.isShowing)
    })
  }
}

// 3. 使用 hdc 查看 Ability 状态
// hdc shell aa dump -a
// hdc shell aa dump -all
```

## 最佳实践

### 1. 合理设计 Ability 结构

- 单一职责：每个 Ability 只负责一个明确的业务功能
- 适度拆分：避免创建过于庞大的 Ability
- 合理复用：通过 Service Ability 和 Data Ability 复用功能

### 2. 优化启动性能

- 延迟加载：在 onWindowStageCreate 后再加载非关键资源
- 预加载：使用预加载减少用户等待时间
- 缓存策略：合理缓存数据避免重复加载

### 3. 确保数据安全

- 权限控制：严格声明和检查权限
- 数据隔离：使用 Data Ability 统一管理敏感数据
- 输入验证：对所有传入参数进行验证

### 4. 处理多设备适配

- 响应式设计：适应不同屏幕尺寸和方向
- 能力检测：使用 Syscap 检测设备支持的功能
- 优雅降级：在不支持的设备上提供替代方案

## 参考

- [华为官方文档 - Ability 开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ability-development)
- [UIAbility 生命周期](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/uiability-lifecycle)
- [Service Ability 开发](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/serviceability-development)
- [Data Ability 开发](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/dataability-development)
- [Ability 间通信](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ability-interaction)