# HarmonyOS 错误码与调试指南

## 通用错误码分类

| 错误码范围 | 模块 | 说明 |
|-----------|------|------|
| 1300001-1300006 | 窗口管理 | 窗口操作相关错误 |
| 1600001-1600010 | Ability管理 | Ability生命周期相关错误 |
| 201-204 | 权限管理 | 权限申请相关错误 |
| 401 | 参数错误 | 接口参数校验失败 |
| 801 | 不支持 | 当前设备不支持该功能 |

## 窗口错误码 (Window Error Codes)

### 1300001 - 重复操作

- **错误信息**: `Repeated operation.`
- **错误描述**: 当进行某些重复操作时，系统会报此错误码
- **可能原因**: 创建的窗口已经存在时，再次创建该窗口
- **解决方案**:
  ```typescript
  // 在创建窗口前，检查该窗口是否已经存在
  try {
    let windowClass = window.getLastWindow(getContext(this))
    // 窗口已存在，复用现有窗口
  } catch (err) {
    // 窗口不存在，可以安全创建
    let newWindow = window.createWindow({
      name: 'floatingWindow',
      windowType: window.WindowType.TYPE_FLOAT
    })
  }
  ```

### 1300002 - 窗口状态异常 ⭐ 最常见

- **错误信息**: `This window state is abnormal.`
- **错误描述**: 当窗口状态异常（未创建或已被销毁）时操作该窗口
- **常见场景**:
  1. 在 `onWindowStageCreate` 中调用 `setWindowBackgroundColor`
  2. 窗口已被销毁后仍然尝试操作
  3. 悬浮窗权限未申请（TYPE_FLOAT类型窗口）
  4. 系统资源不足
- **解决方案**:
  ```typescript
  // ❌ 错误：在 onWindowStageCreate 中操作窗口属性
  onWindowStageCreate(windowStage: window.WindowStage): void {
    windowStage.getMainWindow((err, data) => {
      data.setWindowBackgroundColor('#F5F5F5')  // 抛出1300002错误
    })
  }

  // ✅ 正确：在页面组件中设置背景色
  @Entry
  @Component
  struct Index {
    build() {
      Column() {
        // 页面内容
      }
      .backgroundColor('#F5F5F5')  // 在这里设置
    }
  }
  ```

### 1300003 - 系统服务工作异常

- **错误信息**: `This window manager service works abnormally.`
- **错误描述**: 窗口内部服务没有正常启动
- **解决方案**: 系统服务内部工作异常，请稍候重试，或者重启设备

### 1300005 - WindowStage异常

- **错误信息**: `This window stage is abnormal.`
- **错误描述**: 操作WindowStage时，该WindowStage已被销毁
- **解决方案**:
  ```typescript
  // 在对WindowStage进行操作前，检查该WindowStage是否存在
  if (windowStage && !windowStage.isDestroyed) {
    windowStage.loadContent('pages/Index', ...)
  }
  ```

### 1300006 - 窗口上下文异常

- **错误信息**: `This window context is abnormal.`
- **错误描述**: 操作窗口上下文时，该窗口上下文已被销毁
- **解决方案**: 在对窗口上下文进行操作前，检查其是否已被销毁

## Ability错误码

### 1600001 - Ability不存在

- **错误信息**: `The specified ability does not exist.`
- **解决方案**: 检查 `want` 参数中的 `bundleName` 和 `abilityName` 是否正确

### 1600002 - Ability正在拉起中

- **错误信息**: `Ability is being started.`
- **解决方案**: 等待Ability启动完成，避免重复调用

### 1600003 - 连接Ability失败

- **错误信息**: `Failed to connect to the ability.`
- **解决方案**: 检查Service Ability是否已启动，网络连接是否正常

### 1600004 - 启动Ability失败

- **错误信息**: `Failed to start the ability.`
- **解决方案**: 
  1. 检查目标Ability是否在module.json5中声明
  2. 检查是否有启动权限
  3. 检查目标Ability的exported属性

### 1600005 - 终止Ability失败

- **错误信息**: `Failed to terminate the ability.`
- **解决方案**: 确保当前Ability处于可终止状态

### 1600006 - 连接Service Ability失败

- **错误信息**: `Failed to connect the service ability.`
- **解决方案**: 
  1. 检查Service Ability是否正确声明
  2. 检查Service Ability是否已启动
  3. 验证连接参数

### 1600007 - 连接已存在

- **错误信息**: `The connection already exists.`
- **解决方案**: 断开现有连接后重新连接

### 1600008 - 连接不存在

- **错误信息**: `The connection does not exist.`
- **解决方案**: 先建立连接，再进行操作

### 1600009 - 保存Ability状态失败

- **错误信息**: `Failed to save the ability state.`
- **解决方案**: 检查状态数据是否可序列化

### 1600010 - 迁移Ability失败

- **错误信息**: `Failed to migrate the ability.`
- **解决方案**: 检查目标设备是否支持分布式迁移

## 权限错误码

### 201 - 权限被拒绝

- **错误信息**: `Permission denied.`
- **解决方案**: 
  1. 在module.json5中声明权限
  2. 对于user_grant权限，运行时动态申请
  3. 引导用户到设置中手动开启权限

### 202 - 非系统应用

- **错误信息**: `Not system application.`
- **解决方案**: 某些权限仅限系统应用使用，需要申请特殊权限或更改设计方案

### 203 - 权限未申请

- **错误信息**: `Permission not granted.`
- **解决方案**: 在module.json5的requestPermissions中声明权限

### 204 - 签名不一致

- **错误信息**: `Signature verification failed.`
- **解决方案**: 确保调用方和被调用方使用相同的签名证书

## 参数错误码

### 401 - 参数错误

- **错误信息**: `Parameter error. Possible causes: ...`
- **常见原因**:
  1. 必填参数未传
  2. 参数类型不匹配
  3. 参数值超出有效范围
  4. 参数格式错误
- **解决方案**: 根据错误信息中的提示，检查并修正参数

## 系统能力错误码

### 801 - 不支持该功能

- **错误信息**: `Capability not supported.`
- **解决方案**: 
  1. 使用 `canIUse` 检查系统能力是否可用
  2. 提供降级方案
  ```typescript
  import { canIUse } from '@kit.AbilityKit'

  if (canIUse('SystemCapability.ArkUI.ArkUI.Full')) {
    // 使用完整功能
  } else {
    // 使用降级方案
  }
  ```

## 调试技巧

### 1. 查看完整错误信息

```typescript
// 在回调中打印完整错误对象
windowStage.loadContent('pages/Index', (err, data) => {
  if (err.code) {
    console.error(`错误码: ${err.code}`)
    console.error(`错误信息: ${err.message}`)
    console.error(`完整错误: ${JSON.stringify(err)}`)
  }
})
```

### 2. 使用hilog记录日志

```typescript
import { hilog } from '@kit.PerformanceAnalysisKit'

// 信息日志
hilog.info(0x0000, 'MyTag', '%{public}s', '这是一条信息日志')

// 错误日志
hilog.error(0x0000, 'MyTag', '错误码: %{public}d, 错误信息: %{public}s', err.code, err.message)

// 在DevEco Studio中过滤: tag:MyTag
```

### 3. 常见崩溃排查流程

| 现象 | 排查步骤 |
|------|---------|
| 白屏后闪退 | 1. 检查EntryAbility中是否有窗口操作<br>2. 检查页面组件是否有语法错误<br>3. 检查main_pages.json配置 |
| 构建失败 | 1. 检查module.json5必填字段<br>2. 检查资源文件是否存在<br>3. 检查权限声明是否完整 |
| 运行时异常 | 1. 查看hilog日志<br>2. 检查错误码含义<br>3. 检查API使用方式 |
| ANR无响应 | 1. 检查是否有阻塞主线程的操作<br>2. 检查循环或递归是否死锁<br>3. 检查内存是否泄漏 |

### 4. 使用hdc命令行工具

```bash
# 查看设备日志
hdc hilog | grep MyTag

# 查看特定应用的日志
hdc hilog | grep com.example.myapp

# 清空日志
hdc hilog -g

# 查看设备信息
hdc shell param get const.product.model

# 查看应用进程
hdc shell ps -ef | grep myapp
```

## 参考

- [华为官方文档 - 通用错误码](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/errorcode-universal)
- [窗口错误码](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/errorcode-window)
- [Ability错误码](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/errorcode-ability)
