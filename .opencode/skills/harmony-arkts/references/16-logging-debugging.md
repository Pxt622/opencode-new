# HarmonyOS 日志、调试与定时器开发指南

## 日志系统

### hilog 日志框架

hilog 是 HarmonyOS 提供的高性能日志框架，支持分级日志、日志过滤和性能分析。

#### 基本使用

```typescript
import { hilog } from '@kit.PerformanceAnalysisKit'

// 日志级别常量
const LOG_DOMAIN = 0x0000 // 日志域，范围 0x0~0xFFFF
const LOG_TAG = 'MyApp'   // 日志标签，用于过滤

// 不同级别的日志
hilog.debug(LOG_DOMAIN, LOG_TAG, '%{public}s', '这是一条调试日志')
hilog.info(LOG_DOMAIN, LOG_TAG, '%{public}s', '这是一条信息日志')
hilog.warn(LOG_DOMAIN, LOG_TAG, '%{public}s', '这是一条警告日志')
hilog.error(LOG_DOMAIN, LOG_TAG, '%{public}s', '这是一条错误日志')
hilog.fatal(LOG_DOMAIN, LOG_TAG, '%{public}s', '这是一条致命错误日志')

// 带格式化参数的日志
const userId = 12345
const operation = 'login'
hilog.info(
  LOG_DOMAIN, 
  LOG_TAG, 
  '用户 %{public}d 执行操作: %{public}s', 
  userId, 
  operation
)

// 敏感信息保护：使用 %{private}s 隐藏敏感数据
const password = 'secret123'
hilog.info(
  LOG_DOMAIN,
  LOG_TAG,
  '密码: %{private}s',  // 输出时会被隐藏
  password
)
```

#### 日志格式化说明

| 格式化标识 | 含义 | 示例 |
|-----------|------|------|
| `%{public}s` | 公共字符串，日志中显示完整内容 | `hilog.info(..., '用户名: %{public}s', '张三')` |
| `%{private}s` | 私有字符串，日志中显示 `<private>` | `hilog.info(..., '密码: %{private}s', '123456')` |
| `%{public}d` | 公共整数 | `hilog.info(..., '年龄: %{public}d', 25)` |
| `%{private}d` | 私有整数 | `hilog.info(..., '手机号: %{private}d', 13800138000)` |
| `%{public}f` | 公共浮点数 | `hilog.info(..., '价格: %{public}f', 99.99)` |
| `%{private}f` | 私有浮点数 | `hilog.info(..., '余额: %{private}f', 1000.50)` |

#### 高级日志功能

```typescript
// 1. 性能日志（用于性能分析）
hilog.info(
  0x0000,
  'Performance',
  '函数执行耗时: %{public}d ms',
  performance.now()
)

// 2. 业务日志分类
class BusinessLogger {
  static readonly DOMAIN = 0x1000
  static readonly TAG = 'Business'
  
  static logPayment(amount: number, orderId: string) {
    hilog.info(
      this.DOMAIN,
      this.TAG,
      '支付成功 - 金额: %{public}f, 订单号: %{public}s',
      amount,
      orderId
    )
  }
  
  static logUserAction(userId: string, action: string, details?: any) {
    hilog.info(
      this.DOMAIN,
      this.TAG,
      '用户操作 - 用户ID: %{private}s, 操作: %{public}s, 详情: %{public}s',
      userId,
      action,
      JSON.stringify(details || {})
    )
  }
}

// 3. 错误日志包装器
class ErrorLogger {
  static logError(error: Error, context?: string) {
    hilog.error(
      0x0000,
      'Error',
      '错误: %{public}s, 上下文: %{public}s, 堆栈: %{public}s',
      error.message,
      context || '无上下文',
      error.stack || '无堆栈'
    )
  }
  
  static logApiError(url: string, status: number, error: any) {
    hilog.error(
      0x0000,
      'ApiError',
      'API调用失败 - URL: %{public}s, 状态码: %{public}d, 错误: %{public}s',
      url,
      status,
      JSON.stringify(error)
    )
  }
}

// 使用示例
try {
  // 某些可能出错的操作
  throw new Error('测试错误')
} catch (error) {
  ErrorLogger.logError(error, '测试场景')
}
```

### 日志配置与管理

#### 日志级别控制

```typescript
// 动态调整日志级别
class LogManager {
  private static logLevel: 'debug' | 'info' | 'warn' | 'error' | 'fatal' = 'info'
  
  static setLogLevel(level: 'debug' | 'info' | 'warn' | 'error' | 'fatal') {
    this.logLevel = level
  }
  
  static debug(...args: any[]) {
    if (this.shouldLog('debug')) {
      hilog.debug(0x0000, 'App', '%{public}s', this.formatArgs(args))
    }
  }
  
  static info(...args: any[]) {
    if (this.shouldLog('info')) {
      hilog.info(0x0000, 'App', '%{public}s', this.formatArgs(args))
    }
  }
  
  static error(...args: any[]) {
    if (this.shouldLog('error')) {
      hilog.error(0x0000, 'App', '%{public}s', this.formatArgs(args))
    }
  }
  
  private static shouldLog(level: string): boolean {
    const levels = ['debug', 'info', 'warn', 'error', 'fatal']
    return levels.indexOf(level) >= levels.indexOf(this.logLevel)
  }
  
  private static formatArgs(args: any[]): string {
    return args.map(arg => {
      if (typeof arg === 'object') {
        return JSON.stringify(arg)
      }
      return String(arg)
    }).join(' ')
  }
}

// 使用示例
LogManager.setLogLevel('debug') // 开发环境
LogManager.debug('调试信息', { key: 'value' })
LogManager.info('普通信息')
LogManager.error('错误信息')

LogManager.setLogLevel('error') // 生产环境
LogManager.debug('这条日志不会输出') // 被过滤
LogManager.error('严重错误') // 会输出
```

#### 日志文件管理

```typescript
// 日志文件配置（在 module.json5 中配置）
// {
//   "module": {
//     "requestPermissions": [
//       {
//         "name": "ohos.permission.WRITE_USER_STORAGE"
//       }
//     ]
//   }
// }

import fileio from '@ohos.fileio'

class FileLogger {
  private static logFile: string = '/data/log/myapp.log'
  private static maxFileSize: number = 10 * 1024 * 1024 // 10MB
  
  static async writeToFile(message: string): Promise<void> {
    try {
      // 检查文件大小
      const stats = await fileio.stat(this.logFile)
      if (stats.size > this.maxFileSize) {
        await this.rotateLogFile()
      }
      
      // 写入日志
      const timestamp = new Date().toISOString()
      const logEntry = `[${timestamp}] ${message}\n`
      
      const fd = await fileio.open(this.logFile, fileio.OpenMode.READ_WRITE | fileio.OpenMode.CREATE | fileio.OpenMode.APPEND)
      await fileio.write(fd, logEntry)
      await fileio.close(fd)
    } catch (error) {
      // 文件操作失败，回退到 hilog
      hilog.error(0x0000, 'FileLogger', '写入文件失败: %{public}s', error.message)
    }
  }
  
  private static async rotateLogFile(): Promise<void> {
    const timestamp = new Date().toISOString().replace(/[:.]/g, '-')
    const backupFile = `${this.logFile}.${timestamp}`
    
    try {
      await fileio.copyFile(this.logFile, backupFile)
      await fileio.unlink(this.logFile)
      
      hilog.info(0x0000, 'FileLogger', '日志文件已轮转: %{public}s', backupFile)
    } catch (error) {
      hilog.error(0x0000, 'FileLogger', '日志文件轮转失败: %{public}s', error.message)
    }
  }
  
  static async readLogs(limit: number = 100): Promise<string[]> {
    try {
      const fd = await fileio.open(this.logFile, fileio.OpenMode.READ_ONLY)
      const content = await fileio.readText(fd)
      await fileio.close(fd)
      
      const lines = content.split('\n').filter(line => line.trim())
      return lines.slice(-limit) // 返回最后 limit 行
    } catch (error) {
      hilog.error(0x0000, 'FileLogger', '读取日志失败: %{public}s', error.message)
      return []
    }
  }
}
```

## 调试工具

### hdc 命令行工具

hdc（HarmonyOS Device Connector）是连接和管理 HarmonyOS 设备的命令行工具。

#### 常用命令

```bash
# 查看设备日志
hdc hilog

# 过滤特定标签的日志
hdc hilog | grep MyApp

# 查看特定应用的日志
hdc hilog | grep com.example.myapp

# 查看错误日志
hdc hilog -l E

# 查看警告及以上级别的日志
hdc hilog -l W

# 清空日志
hdc hilog -g

# 保存日志到文件
hdc hilog > log.txt

# 实时查看日志（类似 tail -f）
hdc hilog -f

# 查看设备信息
hdc shell param get const.product.model
hdc shell param get const.product.name
hdc shell param get const.product.manufacturer

# 查看应用信息
hdc shell bm dump -a
hdc shell bm dump -n com.example.myapp

# 查看进程信息
hdc shell ps -ef | grep myapp

# 查看内存使用
hdc shell top -n 1 | grep myapp

# 查看 CPU 使用
hdc shell dumpsys cpuinfo | grep myapp

# 查看网络连接
hdc shell netstat -an | grep ESTABLISHED
```

#### 高级调试命令

```bash
# 性能分析
hdc shell hiprofiler start --app com.example.myapp
hdc shell hiprofiler stop
hdc shell hiprofiler dump

# 内存分析
hdc shell meminfo com.example.myapp

# 查看线程信息
hdc shell ps -t $(pidof com.example.myapp)

# 查看文件描述符
hdc shell ls -la /proc/$(pidof com.example.myapp)/fd

# 查看系统属性
hdc shell getprop

# 查看内核日志
hdc shell dmesg

# 查看电池信息
hdc shell dumpsys battery

# 查看显示信息
hdc shell dumpsys display
```

### DevEco Studio 调试功能

#### 断点调试

```typescript
// 1. 条件断点
function processOrder(orderId: string, amount: number) {
  // 在这里设置条件断点：amount > 1000
  console.log(`处理订单: ${orderId}, 金额: ${amount}`)
  
  // 业务逻辑
  if (amount > 1000) {
    console.log('大额订单，需要审核')
    // 断点可以设置在这里，当 amount > 1000 时触发
  }
}

// 2. 日志断点
class UserService {
  login(username: string, password: string) {
    // 设置日志断点：打印 "用户登录: {username}"
    const token = this.authenticate(username, password)
    
    // 设置日志断点：打印 "认证成功，token: {token}"
    return token
  }
}

// 3. 异常断点
async function fetchData() {
  try {
    // 设置异常断点：捕获所有异常
    const response = await httpRequest('https://api.example.com/data')
    return response.data
  } catch (error) {
    // 异常断点会在这里暂停
    console.error('请求失败:', error)
    throw error
  }
}
```

#### 性能分析器

```typescript
// 性能监控示例
class PerformanceMonitor {
  private static measurements: Map<string, number> = new Map()
  
  static startMeasure(name: string) {
    this.measurements.set(name, performance.now())
  }
  
  static endMeasure(name: string) {
    const startTime = this.measurements.get(name)
    if (startTime) {
      const duration = performance.now() - startTime
      hilog.info(0x0000, 'Performance', '%{public}s: %{public}f ms', name, duration)
      
      // 性能警告
      if (duration > 100) { // 超过100ms
        hilog.warn(0x0000, 'Performance', '性能警告: %{public}s 耗时过长', name)
      }
      
      this.measurements.delete(name)
    }
  }
  
  static async measureAsync<T>(name: string, operation: () => Promise<T>): Promise<T> {
    this.startMeasure(name)
    try {
      const result = await operation()
      this.endMeasure(name)
      return result
    } catch (error) {
      this.endMeasure(name)
      throw error
    }
  }
}

// 使用示例
async function loadUserData() {
  return PerformanceMonitor.measureAsync('loadUserData', async () => {
    // 模拟耗时操作
    await new Promise(resolve => setTimeout(resolve, 200))
    return { id: 1, name: '张三' }
  })
}
```

## 定时器与任务调度

### setTimeout / setInterval

```typescript
// 基础定时器
let timerId: number

// setTimeout - 一次性定时器
timerId = setTimeout(() => {
  console.log('3秒后执行')
}, 3000)

// setInterval - 重复定时器
timerId = setInterval(() => {
  console.log('每隔1秒执行一次')
}, 1000)

// 清除定时器
clearTimeout(timerId)
clearInterval(timerId)
```

### 定时器管理类

```typescript
class TimerManager {
  private timers: Map<string, number> = new Map()
  
  // 创建一次性定时器
  setTimeout(id: string, callback: () => void, delay: number): void {
    this.clearTimer(id)
    
    const timerId = setTimeout(() => {
      callback()
      this.timers.delete(id)
    }, delay)
    
    this.timers.set(id, timerId)
  }
  
  // 创建重复定时器
  setInterval(id: string, callback: () => void, interval: number): void {
    this.clearTimer(id)
    
    const timerId = setInterval(callback, interval)
    this.timers.set(id, timerId)
  }
  
  // 清除定时器
  clearTimer(id: string): void {
    const timerId = this.timers.get(id)
    if (timerId) {
      clearTimeout(timerId)
      clearInterval(timerId)
      this.timers.delete(id)
    }
  }
  
  // 清除所有定时器
  clearAll(): void {
    for (const timerId of this.timers.values()) {
      clearTimeout(timerId)
      clearInterval(timerId)
    }
    this.timers.clear()
  }
  
  // 检查定时器是否存在
  hasTimer(id: string): boolean {
    return this.timers.has(id)
  }
  
  // 获取剩余定时器数量
  getTimerCount(): number {
    return this.timers.size
  }
}

// 使用示例
const timerManager = new TimerManager()

// 创建定时器
timerManager.setTimeout('loginTimeout', () => {
  console.log('登录超时')
}, 30000)

timerManager.setInterval('heartbeat', () => {
  console.log('发送心跳包')
}, 5000)

// 页面销毁时清理
onDestroy() {
  timerManager.clearAll()
}
```

### 后台任务调度

```typescript
import backgroundTaskManager from '@ohos.resourceschedule.backgroundTaskManager'

// 延迟任务
async function scheduleDelayedTask(): Promise<void> {
  try {
    const delayInfo = {
      requestId: 1,
      abilityName: 'com.example.myapp.EntryAbility',
      bundleName: 'com.example.myapp'
    }
    
    // 申请延迟任务权限
    await backgroundTaskManager.requestSuspendDelay(
      '执行后台任务',
      delayInfo
    )
    
    // 执行后台任务
    await performBackgroundTask()
    
    // 任务完成后取消延迟
    backgroundTaskManager.cancelSuspendDelay(delayInfo.requestId)
  } catch (error) {
    console.error('后台任务调度失败:', error)
  }
}

// 持续任务
async function startContinuousTask(): Promise<void> {
  try {
    // 申请持续任务权限（需要在 module.json5 中声明权限）
    await backgroundTaskManager.startBackgroundRunning({
      bundleName: 'com.example.myapp',
      abilityName: 'com.example.myapp.EntryAbility',
      notificationId: 1,
      notificationContent: {
        title: '后台任务运行中',
        text: '正在执行重要任务...'
      }
    })
    
    // 执行持续任务
    while (true) {
      await performContinuousTask()
      await sleep(60000) // 每分钟执行一次
    }
  } catch (error) {
    console.error('持续任务启动失败:', error)
  }
}

// 停止持续任务
async function stopContinuousTask(): Promise<void> {
  try {
    await backgroundTaskManager.stopBackgroundRunning({
      bundleName: 'com.example.myapp',
      abilityName: 'com.example.myapp.EntryAbility'
    })
  } catch (error) {
    console.error('停止持续任务失败:', error)
  }
}

// 辅助函数
async function performBackgroundTask(): Promise<void> {
  // 模拟后台任务
  await new Promise(resolve => setTimeout(resolve, 5000))
  console.log('后台任务完成')
}

async function performContinuousTask(): Promise<void> {
  // 模拟持续任务
  console.log('执行持续任务:', new Date().toISOString())
}

function sleep(ms: number): Promise<void> {
  return new Promise(resolve => setTimeout(resolve, ms))
}
```

### 定时任务框架

```typescript
interface ScheduledTask {
  id: string
  name: string
  interval: number // 执行间隔（毫秒）
  lastRun: number // 上次执行时间
  enabled: boolean
  execute: () => Promise<void>
}

class TaskScheduler {
  private tasks: Map<string, ScheduledTask> = new Map()
  private timerId: number | null = null
  
  // 注册任务
  registerTask(task: ScheduledTask): void {
    this.tasks.set(task.id, {
      ...task,
      lastRun: 0,
      enabled: true
    })
  }
  
  // 启动调度器
  start(): void {
    if (this.timerId) {
      return
    }
    
    this.timerId = setInterval(() => {
      this.runTasks()
    }, 1000) // 每秒检查一次任务
  }
  
  // 停止调度器
  stop(): void {
    if (this.timerId) {
      clearInterval(this.timerId)
      this.timerId = null
    }
  }
  
  // 执行任务
  private async runTasks(): Promise<void> {
    const now = Date.now()
    
    for (const task of this.tasks.values()) {
      if (!task.enabled) {
        continue
      }
      
      if (now - task.lastRun >= task.interval) {
        try {
          hilog.info(0x0000, 'TaskScheduler', '开始执行任务: %{public}s', task.name)
          await task.execute()
          task.lastRun = now
          hilog.info(0x0000, 'TaskScheduler', '任务完成: %{public}s', task.name)
        } catch (error) {
          hilog.error(0x0000, 'TaskScheduler', '任务执行失败: %{public}s, 错误: %{public}s', 
            task.name, error.message)
        }
      }
    }
  }
  
  // 启用/禁用任务
  setTaskEnabled(taskId: string, enabled: boolean): void {
    const task = this.tasks.get(taskId)
    if (task) {
      task.enabled = enabled
      if (enabled) {
        task.lastRun = 0 // 重置上次执行时间
      }
    }
  }
  
  // 手动触发任务
  async triggerTask(taskId: string): Promise<void> {
    const task = this.tasks.get(taskId)
    if (task && task.enabled) {
      try {
        await task.execute()
        task.lastRun = Date.now()
      } catch (error) {
        throw new Error(`任务执行失败: ${error.message}`)
      }
    }
  }
}

// 使用示例
const scheduler = new TaskScheduler()

// 注册数据同步任务
scheduler.registerTask({
  id: 'data_sync',
  name: '数据同步',
  interval: 5 * 60 * 1000, // 每5分钟
  execute: async () => {
    console.log('开始同步数据...')
    // 执行数据同步逻辑
    await new Promise(resolve => setTimeout(resolve, 2000))
    console.log('数据同步完成')
  }
})

// 注册日志上传任务
scheduler.registerTask({
  id: 'log_upload',
  name: '日志上传',
  interval: 30 * 60 * 1000, // 每30分钟
  execute: async () => {
    console.log('开始上传日志...')
    // 执行日志上传逻辑
    await new Promise(resolve => setTimeout(resolve, 1000))
    console.log('日志上传完成')
  }
})

// 启动调度器
scheduler.start()

// 应用退出时停止
onDestroy() {
  scheduler.stop()
}
```

## 错误监控与上报

### 全局错误处理器

```typescript
// 全局错误捕获
class GlobalErrorHandler {
  static init(): void {
    // 捕获未处理的 Promise 异常
    process.on('unhandledRejection', (reason, promise) => {
      hilog.error(0x0000, 'GlobalError', 
        '未处理的 Promise 异常: %{public}s, Promise: %{public}s', 
        reason, promise)
      
      // 上报错误
      this.reportError({
        type: 'unhandledRejection',
        reason: String(reason),
        timestamp: Date.now()
      })
    })
    
    // 捕获未捕获的异常
    process.on('uncaughtException', (error) => {
      hilog.fatal(0x0000, 'GlobalError', 
        '未捕获的异常: %{public}s, 堆栈: %{public}s', 
        error.message, error.stack)
      
      // 上报错误
      this.reportError({
        type: 'uncaughtException',
        message: error.message,
        stack: error.stack,
        timestamp: Date.now()
      })
    })
  }
  
  // 错误上报
  static async reportError(errorInfo: any): Promise<void> {
    try {
      // 保存到本地
      await this.saveErrorToLocal(errorInfo)
      
      // 如果有网络，上报到服务器
      if (await this.checkNetwork()) {
        await this.uploadErrorToServer(errorInfo)
      }
    } catch (error) {
      hilog.error(0x0000, 'ErrorHandler', '错误上报失败: %{public}s', error.message)
    }
  }
  
  // 保存错误到本地
  private static async saveErrorToLocal(errorInfo: any): Promise<void> {
    const errors = await this.getStoredErrors()
    errors.push({
      ...errorInfo,
      id: Date.now() + Math.random().toString(36).substr(2, 9)
    })
    
    // 只保存最近的100个错误
    if (errors.length > 100) {
      errors.splice(0, errors.length - 100)
    }
    
    // 保存到 Preferences
    const preferences = await dataPreferences.getPreferences(getContext(), 'error_logs')
    await preferences.put('errors', JSON.stringify(errors))
    await preferences.flush()
  }
  
  // 获取存储的错误
  private static async getStoredErrors(): Promise<any[]> {
    try {
      const preferences = await dataPreferences.getPreferences(getContext(), 'error_logs')
      const errorsJson = await preferences.get('errors', '[]')
      return JSON.parse(errorsJson)
    } catch (error) {
      return []
    }
  }
  
  // 检查网络
  private static async checkNetwork(): Promise<boolean> {
    // 简化的网络检查
    return navigator.onLine || false
  }
  
  // 上传错误到服务器
  private static async uploadErrorToServer(errorInfo: any): Promise<void> {
    // 这里应该实现实际上传逻辑
    console.log('上传错误到服务器:', errorInfo)
    
    // 上传成功后删除本地记录
    const preferences = await dataPreferences.getPreferences(getContext(), 'error_logs')
    await preferences.delete('errors')
    await preferences.flush()
  }
}

// 初始化全局错误处理器
GlobalErrorHandler.init()
```

## 性能调试与跟踪

### HiTrace 性能跟踪

HiTrace 是 HarmonyOS 提供的性能跟踪工具，用于分析应用性能瓶颈和系统级问题。

#### HiTraceMeter 使用

```typescript
import { hiTraceMeter } from '@kit.PerformanceAnalysisKit'

// 同步任务性能跟踪
function traceSyncOperation() {
  // 开始跟踪，指定任务名称
  hiTraceMeter.startTrace('loadUserData', 1001)
  
  try {
    // 执行业务逻辑
    const userData = loadUserDataFromDatabase()
    processUserData(userData)
    
    // 结束跟踪
    hiTraceMeter.finishTrace('loadUserData', 1001)
  } catch (error) {
    hiTraceMeter.finishTrace('loadUserData', 1001)
    throw error
  }
}

// 异步任务性能跟踪
async function traceAsyncOperation() {
  hiTraceMeter.startTrace('fetchRemoteData', 1002)
  
  try {
    const response = await fetchDataFromServer()
    await processResponse(response)
    
    hiTraceMeter.finishTrace('fetchRemoteData', 1002)
  } catch (error) {
    hiTraceMeter.finishTrace('fetchRemoteData', 1002)
    throw error
  }
}

// 计数器跟踪（用于跟踪数值变化）
function traceCounterMetrics() {
  // 记录内存使用
  const memoryUsage = getMemoryUsage()
  hiTraceMeter.traceByValue('memoryUsage', memoryUsage)
  
  // 记录CPU使用率
  const cpuUsage = getCpuUsage()
  hiTraceMeter.traceByValue('cpuUsage', cpuUsage)
  
  // 记录帧率
  const fps = getCurrentFps()
  hiTraceMeter.traceByValue('fps', fps)
}

// 嵌套跟踪
function traceNestedOperations() {
  hiTraceMeter.startTrace('parentOperation', 2001)
  
  // 子任务1
  hiTraceMeter.startTrace('childTask1', 2002)
  performChildTask1()
  hiTraceMeter.finishTrace('childTask1', 2002)
  
  // 子任务2
  hiTraceMeter.startTrace('childTask2', 2003)
  performChildTask2()
  hiTraceMeter.finishTrace('childTask2', 2003)
  
  hiTraceMeter.finishTrace('parentOperation', 2001)
}
```

#### HiTraceChain 使用

```typescript
import { hiTraceChain } from '@kit.PerformanceAnalysisKit'

// 开始跟踪链
function startTraceChain(name: string): hiTraceChain.HiTraceId {
  const traceId = hiTraceChain.begin(name, hiTraceChain.HiTraceFlag.INCLUDE_ASYNC)
  console.log(`开始跟踪链: ${name}, ID: ${traceId.getChainId()}`)
  return traceId
}

// 结束跟踪链
function endTraceChain(traceId: hiTraceChain.HiTraceId) {
  hiTraceChain.end(traceId)
  console.log(`结束跟踪链, ID: ${traceId.getChainId()}`)
}

// 跨进程/跨线程传递跟踪上下文
async function crossProcessTrace() {
  const traceId = startTraceChain('userLogin')
  
  try {
    // 将跟踪ID传递给其他进程或服务
    const traceIdString = traceId.toString()
    
    // 调用远程服务时传递跟踪信息
    await callRemoteService({
      data: loginData,
      traceId: traceIdString
    })
    
    endTraceChain(traceId)
  } catch (error) {
    endTraceChain(traceId)
    throw error
  }
}

// 从字符串恢复跟踪上下文
function restoreTraceContext(traceIdString: string): hiTraceChain.HiTraceId {
  return hiTraceChain.parse(traceIdString)
}

// 在子任务中使用跟踪上下文
function handleSubTask(parentTraceId: hiTraceChain.HiTraceId) {
  // 创建子跟踪
  const childTraceId = hiTraceChain.createSpan()
  
  hiTraceChain.beginSpan('subTask', childTraceId)
  
  try {
    // 执行业务逻辑
    processSubTask()
    
    hiTraceChain.endSpan(childTraceId)
  } catch (error) {
    hiTraceChain.endSpan(childTraceId)
    throw error
  }
}
```

#### HiTraceMeter 高级用法

```typescript
import { hiTraceMeter } from '@kit.PerformanceAnalysisKit'

// 性能监控类
class PerformanceTracer {
  private traceMap: Map<string, number> = new Map()
  
  // 开始跟踪任务
  startTrace(taskName: string): void {
    const traceId = this.generateTraceId()
    this.traceMap.set(taskName, traceId)
    hiTraceMeter.startTrace(taskName, traceId)
  }
  
  // 结束跟踪任务
  endTrace(taskName: string): void {
    const traceId = this.traceMap.get(taskName)
    if (traceId !== undefined) {
      hiTraceMeter.finishTrace(taskName, traceId)
      this.traceMap.delete(taskName)
    }
  }
  
  // 跟踪函数执行时间
  traceFunction<T>(name: string, fn: () => T): T {
    this.startTrace(name)
    try {
      const result = fn()
      this.endTrace(name)
      return result
    } catch (error) {
      this.endTrace(name)
      throw error
    }
  }
  
  // 跟踪异步函数执行时间
  async traceAsyncFunction<T>(name: string, fn: () => Promise<T>): Promise<T> {
    this.startTrace(name)
    try {
      const result = await fn()
      this.endTrace(name)
      return result
    } catch (error) {
      this.endTrace(name)
      throw error
    }
  }
  
  // 跟踪计数器值
  traceCounter(name: string, value: number): void {
    hiTraceMeter.traceByValue(name, value)
  }
  
  private generateTraceId(): number {
    return Date.now() + Math.floor(Math.random() * 1000)
  }
}

// 使用示例
const tracer = new PerformanceTracer()

// 跟踪同步函数
const result = tracer.traceFunction('calculateData', () => {
  // 计算逻辑
  return computeExpensiveOperation()
})

// 跟踪异步函数
await tracer.traceAsyncFunction('fetchData', async () => {
  const data = await fetchRemoteData()
  return processData(data)
})

// 跟踪计数器
setInterval(() => {
  tracer.traceCounter('memoryUsage', getMemoryUsage())
  tracer.traceCounter('cpuUsage', getCpuUsage())
}, 5000)
```

### 性能检测 API

```typescript
import { performance } from '@kit.PerformanceAnalysisKit'

// 获取应用启动时间
function getAppStartupTime(): number {
  // 获取应用启动时间戳
  const startupTime = performance.getAppStartupTime()
  console.log(`应用启动时间: ${startupTime}ms`)
  return startupTime
}

// 获取页面加载时间
function getPageLoadTime(): number {
  const pageLoadTime = performance.getPageLoadTime()
  console.log(`页面加载时间: ${pageLoadTime}ms`)
  return pageLoadTime
}

// 性能指标监控
class PerformanceMonitor {
  private metrics: Map<string, number[]> = new Map()
  
  // 记录性能指标
  recordMetric(name: string, value: number): void {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, [])
    }
    this.metrics.get(name)!.push(value)
  }
  
  // 获取平均性能指标
  getAverageMetric(name: string): number {
    const values = this.metrics.get(name) || []
    if (values.length === 0) return 0
    
    const sum = values.reduce((a, b) => a + b, 0)
    return sum / values.length
  }
  
  // 获取性能统计
  getMetricStats(name: string): { min: number; max: number; avg: number; count: number } {
    const values = this.metrics.get(name) || []
    if (values.length === 0) {
      return { min: 0, max: 0, avg: 0, count: 0 }
    }
    
    const min = Math.min(...values)
    const max = Math.max(...values)
    const avg = values.reduce((a, b) => a + b, 0) / values.length
    
    return { min, max, avg, count: values.length }
  }
  
  // 清除指标数据
  clearMetrics(): void {
    this.metrics.clear()
  }
}

// 使用示例
const monitor = new PerformanceMonitor()

// 监控页面加载性能
async function loadPage() {
  const startTime = performance.now()
  
  await renderPage()
  
  const endTime = performance.now()
  const loadTime = endTime - startTime
  
  monitor.recordMetric('pageLoadTime', loadTime)
  console.log(`页面加载耗时: ${loadTime}ms`)
}

// 获取性能统计
const stats = monitor.getMetricStats('pageLoadTime')
console.log('页面加载性能统计:', stats)
```

### 应用冻结检测 (AppFreeze)

```typescript
import { appFreeze } from '@kit.PerformanceAnalysisKit'

// 注册应用冻结监听器
class AppFreezeDetector {
  private listenerId: number = 0
  
  // 开始监听应用冻结
  startMonitoring(): void {
    this.listenerId = appFreeze.on('appFreeze', (info) => {
      console.warn('检测到应用冻结:', info)
      
      // 记录冻结信息
      this.handleAppFreeze(info)
    })
    
    console.log('应用冻结检测已启动')
  }
  
  // 停止监听
  stopMonitoring(): void {
    if (this.listenerId !== 0) {
      appFreeze.off('appFreeze', this.listenerId)
      this.listenerId = 0
      console.log('应用冻结检测已停止')
    }
  }
  
  // 处理应用冻结
  private handleAppFreeze(info: appFreeze.AppFreezeInfo): void {
    // 记录到日志
    hilog.error(
      0x0000,
      'AppFreeze',
      '应用冻结检测 - 类型: %{public}s, 时间: %{public}s, 详情: %{public}s',
      info.type,
      new Date(info.timestamp).toISOString(),
      JSON.stringify(info.details)
    )
    
    // 上报到服务器
    this.reportFreeze(info)
    
    // 尝试恢复
    this.attemptRecovery(info)
  }
  
  // 上报冻结信息
  private async reportFreeze(info: appFreeze.AppFreezeInfo): Promise<void> {
    try {
      // 实现上报逻辑
      await uploadFreezeReport(info)
    } catch (error) {
      console.error('上报冻结信息失败:', error)
    }
  }
  
  // 尝试恢复应用
  private attemptRecovery(info: appFreeze.AppFreezeInfo): void {
    // 根据冻结类型采取不同的恢复策略
    switch (info.type) {
      case 'main_thread_blocked':
        // 主线程阻塞，尝试清理任务
        console.warn('主线程阻塞，尝试清理后台任务')
        break
      case 'input_not_responding':
        // 输入无响应
        console.warn('输入无响应，尝试恢复输入处理')
        break
      case 'launch_timeout':
        // 启动超时
        console.warn('启动超时，尝试优化启动流程')
        break
    }
  }
}

// 使用示例
const freezeDetector = new AppFreezeDetector()
freezeDetector.startMonitoring()

// 页面销毁时停止监听
onDestroy() {
  freezeDetector.stopMonitoring()
}
```

### 资源泄漏检测

```typescript
import { leakDetector } from '@kit.PerformanceAnalysisKit'

// 资源泄漏检测配置
interface LeakDetectionConfig {
  enableMemoryLeak: boolean      // 启用内存泄漏检测
  enableResourceLeak: boolean    // 启用资源泄漏检测
  checkInterval: number          // 检查间隔（毫秒）
  threshold: number              // 泄漏阈值
}

// 资源泄漏检测器
class ResourceLeakDetector {
  private config: LeakDetectionConfig
  private checkTimer: number = 0
  
  constructor(config: LeakDetectionConfig) {
    this.config = config
  }
  
  // 启动泄漏检测
  startDetection(): void {
    if (this.checkTimer) {
      return
    }
    
    this.checkTimer = setInterval(() => {
      this.checkForLeaks()
    }, this.config.checkInterval)
    
    console.log('资源泄漏检测已启动')
  }
  
  // 停止泄漏检测
  stopDetection(): void {
    if (this.checkTimer) {
      clearInterval(this.checkTimer)
      this.checkTimer = 0
      console.log('资源泄漏检测已停止')
    }
  }
  
  // 检查泄漏
  private checkForLeaks(): void {
    if (this.config.enableMemoryLeak) {
      this.checkMemoryLeak()
    }
    
    if (this.config.enableResourceLeak) {
      this.checkResourceLeak()
    }
  }
  
  // 检查内存泄漏
  private checkMemoryLeak(): void {
    try {
      const memoryInfo = leakDetector.getMemoryInfo()
      
      if (memoryInfo.heapUsed > this.config.threshold) {
        hilog.warn(
          0x0000,
          'LeakDetector',
          '检测到内存使用过高: %{public}d bytes, 阈值: %{public}d bytes',
          memoryInfo.heapUsed,
          this.config.threshold
        )
        
        // 触发内存分析
        this.analyzeMemoryUsage()
      }
    } catch (error) {
      console.error('内存泄漏检测失败:', error)
    }
  }
  
  // 检查资源泄漏
  private checkResourceLeak(): void {
    try {
      const resourceInfo = leakDetector.getResourceInfo()
      
      if (resourceInfo.unclosedHandles > 10) {
        hilog.warn(
          0x0000,
          'LeakDetector',
          '检测到未关闭资源句柄: %{public}d',
          resourceInfo.unclosedHandles
        )
      }
      
      if (resourceInfo.unreleasedObjects > 50) {
        hilog.warn(
          0x0000,
          'LeakDetector',
          '检测到未释放对象: %{public}d',
          resourceInfo.unreleasedObjects
        )
      }
    } catch (error) {
      console.error('资源泄漏检测失败:', error)
    }
  }
  
  // 分析内存使用
  private analyzeMemoryUsage(): void {
    try {
      const snapshot = leakDetector.takeHeapSnapshot()
      
      hilog.info(
        0x0000,
        'LeakDetector',
        '内存快照 - 总大小: %{public}d, 对象数: %{public}d',
        snapshot.totalSize,
        snapshot.objectCount
      )
      
      // 分析大对象
      const largeObjects = snapshot.objects.filter(obj => obj.size > 1024 * 1024)
      if (largeObjects.length > 0) {
        hilog.warn(
          0x0000,
          'LeakDetector',
          '发现大对象: %{public}d 个',
          largeObjects.length
        )
        
        largeObjects.forEach(obj => {
          hilog.info(
            0x0000,
            'LeakDetector',
            '大对象 - 类型: %{public}s, 大小: %{public}d bytes',
            obj.type,
            obj.size
          )
        })
      }
    } catch (error) {
      console.error('内存分析失败:', error)
    }
  }
}

// 使用示例
const leakDetector = new ResourceLeakDetector({
  enableMemoryLeak: true,
  enableResourceLeak: true,
  checkInterval: 30000,  // 每30秒检查一次
  threshold: 100 * 1024 * 1024  // 100MB阈值
})

leakDetector.startDetection()

// 页面销毁时停止
onDestroy() {
  leakDetector.stopDetection()
}
```

### 电源检测

```typescript
import { battery } from '@ohos.battery'
import { power } from '@kit.PerformanceAnalysisKit'

// 电源状态检测
class PowerMonitor {
  private batteryLevel: number = 100
  private isCharging: boolean = false
  private batteryStatus: battery.BatteryState = battery.BatteryState.FULL
  
  // 获取电池信息
  async getBatteryInfo(): Promise<battery.BatteryInfo> {
    try {
      const info = await battery.getBatteryInfo()
      
      this.batteryLevel = info.level
      this.isCharging = info.charging
      this.batteryStatus = info.state
      
      return info
    } catch (error) {
      console.error('获取电池信息失败:', error)
      throw error
    }
  }
  
  // 监听电池变化
  registerBatteryListener(): void {
    battery.on('batteryChange', (info) => {
      this.batteryLevel = info.level
      this.isCharging = info.charging
      this.batteryStatus = info.state
      
      this.handleBatteryChange(info)
    })
  }
  
  // 处理电池变化
  private handleBatteryChange(info: battery.BatteryInfo): void {
    hilog.info(
      0x0000,
      'PowerMonitor',
      '电池状态变化 - 电量: %{public}d%%, 充电状态: %{public}s',
      info.level,
      info.charging ? '充电中' : '未充电'
    )
    
    // 低电量警告
    if (info.level <= 20 && !info.charging) {
      hilog.warn(
        0x0000,
        'PowerMonitor',
        '低电量警告: %{public}d%%',
        info.level
      )
      
      // 触发低电量处理
      this.handleLowBattery(info.level)
    }
    
    // 电量充足
    if (info.level >= 80 && info.charging) {
      hilog.info(
        0x0000,
        'PowerMonitor',
        '电量充足: %{public}d%%',
        info.level
      )
    }
  }
  
  // 低电量处理
  private handleLowBattery(level: number): void {
    // 降低后台任务频率
    reduceBackgroundTaskFrequency()
    
    // 关闭非必要功能
    disableNonEssentialFeatures()
    
    // 提示用户
    showLowBatteryWarning(level)
  }
  
  // 获取电源策略
  getPowerStrategy(): string {
    if (this.isCharging) {
      return 'high_performance'
    } else if (this.batteryLevel > 50) {
      return 'balanced'
    } else if (this.batteryLevel > 20) {
      return 'power_save'
    } else {
      return 'ultra_power_save'
    }
  }
  
  // 检查是否允许后台任务
  isBackgroundTaskAllowed(): boolean {
    if (this.isCharging) {
      return true
    }
    return this.batteryLevel > 30
  }
}

// 使用示例
const powerMonitor = new PowerMonitor()

// 获取当前电池信息
const batteryInfo = await powerMonitor.getBatteryInfo()
console.log(`电量: ${batteryInfo.level}%, 充电状态: ${batteryInfo.charging}`)

// 注册监听
powerMonitor.registerBatteryListener()

// 根据电源策略调整应用行为
const strategy = powerMonitor.getPowerStrategy()
console.log('当前电源策略:', strategy)
```

### 应用任务超时检测

```typescript
import { taskTimeout } from '@kit.PerformanceAnalysisKit'

// 任务超时配置
interface TaskTimeoutConfig {
  timeout: number        // 超时时间（毫秒）
  retryCount: number     // 重试次数
  retryDelay: number     // 重试延迟（毫秒）
}

// 带超时控制的任务执行器
class TimeoutTaskExecutor {
  private config: TaskTimeoutConfig
  
  constructor(config: TaskTimeoutConfig) {
    this.config = config
  }
  
  // 执行任务（带超时控制）
  async executeWithTimeout<T>(
    task: () => Promise<T>,
    taskName: string,
    customTimeout?: number
  ): Promise<T> {
    const timeout = customTimeout || this.config.timeout
    
    return new Promise(async (resolve, reject) => {
      // 创建超时定时器
      const timeoutId = setTimeout(() => {
        reject(new Error(`任务 "${taskName}" 执行超时 (${timeout}ms)`))
      }, timeout)
      
      try {
        const result = await task()
        clearTimeout(timeoutId)
        resolve(result)
      } catch (error) {
        clearTimeout(timeoutId)
        reject(error)
      }
    })
  }
  
  // 执行任务（带重试机制）
  async executeWithRetry<T>(
    task: () => Promise<T>,
    taskName: string,
    retryCount?: number
  ): Promise<T> {
    const maxRetries = retryCount !== undefined ? retryCount : this.config.retryCount
    let lastError: Error | null = null
    
    for (let i = 0; i <= maxRetries; i++) {
      try {
        if (i > 0) {
          hilog.info(
            0x0000,
            'TaskExecutor',
            '任务 "%{public}s" 第 %{public}d 次重试',
            taskName,
            i
          )
          
          // 等待重试延迟
          await this.delay(this.config.retryDelay * i)
        }
        
        return await this.executeWithTimeout(task, taskName)
      } catch (error) {
        lastError = error as Error
        
        hilog.error(
          0x0000,
          'TaskExecutor',
          '任务 "%{public}s" 执行失败 (第 %{public}d 次): %{public}s',
          taskName,
          i + 1,
          lastError.message
        )
      }
    }
    
    throw new Error(
      `任务 "${taskName}" 在 ${maxRetries} 次重试后仍然失败: ${lastError?.message}`
    )
  }
  
  // 批量执行任务（带并发控制）
  async executeBatch<T>(
    tasks: Array<{ name: string; task: () => Promise<T> }>,
    concurrency: number = 3
  ): Promise<Array<{ name: string; result?: T; error?: Error }>> {
    const results: Array<{ name: string; result?: T; error?: Error }> = []
    const executing: Promise<void>[] = []
    
    for (const { name, task } of tasks) {
      const promise = this.executeWithTimeout(task, name)
        .then(result => {
          results.push({ name, result })
        })
        .catch(error => {
          results.push({ name, error: error as Error })
        })
        .finally(() => {
          // 从执行队列中移除
          const index = executing.indexOf(promise)
          if (index > -1) {
            executing.splice(index, 1)
          }
        })
      
      executing.push(promise)
      
      // 控制并发数
      if (executing.length >= concurrency) {
        await Promise.race(executing)
      }
    }
    
    // 等待所有任务完成
    await Promise.all(executing)
    
    return results
  }
  
  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }
}

// 使用示例
const executor = new TimeoutTaskExecutor({
  timeout: 10000,    // 10秒超时
  retryCount: 3,     // 重试3次
  retryDelay: 1000   // 重试延迟1秒
})

// 执行单个任务
try {
  const result = await executor.executeWithTimeout(
    () => fetchDataFromServer(),
    'fetchData',
    5000  // 5秒超时
  )
  console.log('任务执行成功:', result)
} catch (error) {
  console.error('任务执行失败:', error)
}

// 执行带重试的任务
try {
  const result = await executor.executeWithRetry(
    () => uploadLargeFile(),
    'uploadFile',
    5  // 最多重试5次
  )
  console.log('上传成功:', result)
} catch (error) {
  console.error('上传失败:', error)
}

// 批量执行任务
const tasks = [
  { name: 'task1', task: () => fetchData1() },
  { name: 'task2', task: () => fetchData2() },
  { name: 'task3', task: () => fetchData3() },
  { name: 'task4', task: () => fetchData4() }
]

const results = await executor.executeBatch(tasks, 2)  // 最多2个并发
results.forEach(({ name, result, error }) => {
  if (error) {
    console.error(`任务 ${name} 失败:`, error)
  } else {
    console.log(`任务 ${name} 成功:`, result)
  }
})
```

### 应用被杀检测

```typescript
import { appKilled } from '@kit.PerformanceAnalysisKit'

// 应用被杀原因类型
type AppKillReason = 
  | 'low_memory'      // 内存不足
  | 'battery_saver'   // 省电模式
  | 'user_action'     // 用户操作
  | 'system_update'   // 系统更新
  | 'crash'           // 崩溃
  | 'timeout'         // 超时
  | 'unknown'         // 未知原因

// 应用被杀信息
interface AppKillInfo {
  reason: AppKillReason
  timestamp: number
  details: string
}

// 应用被杀检测器
class AppKilledDetector {
  private listenerId: number = 0
  private killHistory: AppKillInfo[] = []
  
  // 开始监听
  startMonitoring(): void {
    this.listenerId = appKilled.on('appKilled', (info) => {
      const killInfo: AppKillInfo = {
        reason: info.reason as AppKillReason,
        timestamp: Date.now(),
        details: info.details || ''
      }
      
      this.killHistory.push(killInfo)
      this.handleAppKilled(killInfo)
    })
    
    console.log('应用被杀检测已启动')
  }
  
  // 停止监听
  stopMonitoring(): void {
    if (this.listenerId !== 0) {
      appKilled.off('appKilled', this.listenerId)
      this.listenerId = 0
      console.log('应用被杀检测已停止')
    }
  }
  
  // 处理应用被杀事件
  private handleAppKilled(info: AppKillInfo): void {
    hilog.error(
      0x0000,
      'AppKilled',
      '应用被杀 - 原因: %{public}s, 时间: %{public}s, 详情: %{public}s',
      info.reason,
      new Date(info.timestamp).toISOString(),
      info.details
    )
    
    // 根据原因采取不同措施
    switch (info.reason) {
      case 'low_memory':
        this.handleLowMemoryKill()
        break
      case 'battery_saver':
        this.handleBatterySaverKill()
        break
      case 'user_action':
        this.handleUserActionKill()
        break
      case 'crash':
        this.handleCrashKill(info)
        break
      default:
        this.handleUnknownKill(info)
    }
  }
  
  // 处理内存不足被杀
  private handleLowMemoryKill(): void {
    hilog.warn(
      0x0000,
      'AppKilled',
      '应用因内存不足被系统杀死，建议优化内存使用'
    )
    
    // 记录内存使用报告
    this.logMemoryUsage()
  }
  
  // 处理省电模式被杀
  private handleBatterySaverKill(): void {
    hilog.warn(
      0x0000,
      'AppKilled',
      '应用因省电模式被杀死，建议优化后台任务'
    )
  }
  
  // 处理用户操作被杀
  private handleUserActionKill(): void {
    hilog.info(
      0x0000,
      'AppKilled',
      '应用被用户手动关闭'
    )
  }
  
  // 处理崩溃被杀
  private handleCrashKill(info: AppKillInfo): void {
    hilog.error(
      0x0000,
      'AppKilled',
      '应用因崩溃被杀死: %{public}s',
      info.details
    )
    
    // 上报崩溃信息
    this.reportCrash(info)
  }
  
  // 处理未知原因被杀
  private handleUnknownKill(info: AppKillInfo): void {
    hilog.warn(
      0x0000,
      'AppKilled',
      '应用因未知原因被杀死: %{public}s',
      info.details
    )
  }
  
  // 记录内存使用
  private async logMemoryUsage(): Promise<void> {
    try {
      const memoryInfo = appKilled.getMemoryInfo()
      
      hilog.info(
        0x0000,
        'AppKilled',
        '内存使用报告 - 总内存: %{public}d, 已用: %{public}d, 可用: %{public}d',
        memoryInfo.total,
        memoryInfo.used,
        memoryInfo.available
      )
    } catch (error) {
      console.error('获取内存信息失败:', error)
    }
  }
  
  // 上报崩溃信息
  private async reportCrash(info: AppKillInfo): Promise<void> {
    try {
      // 实现崩溃上报逻辑
      await uploadCrashReport({
        reason: info.reason,
        timestamp: info.timestamp,
        details: info.details,
        memoryInfo: await appKilled.getMemoryInfo()
      })
    } catch (error) {
      console.error('上报崩溃信息失败:', error)
    }
  }
  
  // 获取被杀历史
  getKillHistory(): AppKillInfo[] {
    return [...this.killHistory]
  }
  
  // 清空历史记录
  clearHistory(): void {
    this.killHistory = []
  }
}

// 使用示例
const killDetector = new AppKilledDetector()
killDetector.startMonitoring()

// 页面销毁时停止监听
onDestroy() {
  killDetector.stopMonitoring()
}

// 获取被杀历史
const history = killDetector.getKillHistory()
console.log('应用被杀历史:', history)
```

## 最佳实践

### 1. 日志最佳实践

- **分级记录**：根据重要性使用不同的日志级别
- **敏感信息保护**：使用 `%{private}s` 保护用户隐私
- **结构化日志**：使用 JSON 格式记录结构化数据
- **性能考虑**：生产环境减少 debug 日志，避免日志过多影响性能
- **日志轮转**：定期清理旧日志，避免存储空间不足

### 2. 调试最佳实践

- **开发环境详细日志**：开发时开启 debug 级别日志
- **生产环境精简日志**：生产环境只记录 error 和重要 info 日志
- **使用标签过滤**：为不同模块使用不同的日志标签
- **远程日志收集**：实现日志上传功能，便于问题排查
- **性能监控**：关键操作添加性能监控点

### 3. 定时器最佳实践

- **及时清理**：组件销毁时清理定时器，避免内存泄漏
- **避免嵌套**：避免在定时器回调中创建新的定时器
- **合理间隔**：根据业务需求设置合理的执行间隔
- **错误处理**：定时器任务添加异常捕获
- **资源释放**：长时间任务考虑使用后台任务管理

### 4. 错误处理最佳实践

- **全局捕获**：设置全局错误处理器
- **分级上报**：根据错误严重性分级上报
- **用户友好**：向用户显示友好的错误信息
- **快速恢复**：实现错误后的恢复机制
- **监控告警**：设置错误监控和告警机制

### 5. 性能调试最佳实践

- **使用 HiTraceMeter**：对关键操作进行性能跟踪
- **使用 HiTraceChain**：对跨进程/跨线程操作进行链路跟踪
- **设置性能阈值**：为关键指标设置合理的性能阈值
- **定期性能分析**：定期进行性能分析，发现潜在问题
- **内存泄漏检测**：启用内存泄漏检测，及时发现内存问题
- **电源优化**：根据电池状态调整应用行为
- **任务超时控制**：为异步任务设置合理的超时时间
- **应用被杀分析**：记录应用被杀原因，优化应用稳定性

## 参考

- [华为官方文档 - hilog 日志框架](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-hilog)
- [华为官方文档 - 后台任务管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-resourceschedule-backgroundtaskmanager)
- [华为官方文档 - 定时器 API](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-timer)
- [调试技巧与工具使用](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/debugging-tools)