# HarmonyOS 性能分析与调试进阶指南

## 性能调试体系概述

HarmonyOS 提供了完整的性能分析和调试工具链，帮助开发者定位和解决应用性能问题。

| 工具 | 功能 | 适用场景 |
|------|------|---------|
| **HiTraceMeter** | 代码插桩性能测量 | 函数耗时、关键路径分析 |
| **HiTrace** | 函数调用链路追踪 | 跨模块调用分析、性能瓶颈定位 |
| **Perf Detection** | 运行时性能监控 | 卡顿、内存泄漏、异常检测 |
| **App Freeze Detection** | 应用冻结检测 | ANR、主线程阻塞检测 |
| **Power Detection** | 功耗检测 | 电池消耗、唤醒次数分析 |
| **Resource Leak Detection** | 资源泄漏检测 | 句柄、文件、内存泄漏检测 |

## HiTraceMeter 代码插桩性能测量

### 基础用法

```typescript
import hiTraceMeter from '@ohos.hiTraceMeter'

// 1. 定义性能测量点
@Entry
@Component
struct PerformanceTest {
  @State startTime: number = 0
  @State duration: number = 0

  build() {
    Column() {
      Button('开始测量')
        .onClick(() => {
          // 标记测量开始点
          hiTraceMeter.startTrace('testOperation')
          this.startTime = Date.now()
        })

      Button('结束测量')
        .onClick(() => {
          // 标记测量结束点
          hiTraceMeter.finishTrace('testOperation', false)
          this.duration = Date.now() - this.startTime
          console.log(`操作耗时: ${this.duration}ms`)
        })
    }
  }
}
```

### 高级用法

```typescript
// 2. 测量函数执行时间
function measureFunction<T>(
  name: string,
  func: () => T
): T {
  hiTraceMeter.startTrace(name)
  try {
    return func()
  } finally {
    hiTraceMeter.finishTrace(name, false)
  }
}

// 使用示例
const result = measureFunction('calculateSum', () => {
  let sum = 0
  for (let i = 0; i < 1000000; i++) {
    sum += i
  }
  return sum
})

// 3. 异步函数测量
async function measureAsyncFunction<T>(
  name: string,
  func: () => Promise<T>
): Promise<T> {
  hiTraceMeter.startTrace(name)
  try {
    return await func()
  } finally {
    hiTraceMeter.finishTrace(name, false)
  }
}

// 使用示例
const data = await measureAsyncFunction('fetchData', async () => {
  const response = await fetch('https://api.example.com/data')
  return response.json()
})

// 4. 测量多次操作的平均耗时
class PerformanceBenchmark {
  private static measurements: Map<string, number[]> = new Map()

  static async benchmark<T>(
    name: string,
    func: () => T,
    iterations: number = 10
  ): Promise<BenchmarkResult> {
    const times: number[] = []
    
    for (let i = 0; i < iterations; i++) {
      const start = Date.now()
      func()
      times.push(Date.now() - start)
    }
    
    // 计算统计数据
    const sum = times.reduce((a, b) => a + b, 0)
    const avg = sum / times.length
    const sortedTimes = [...times].sort((a, b) => a - b)
    const min = sortedTimes[0]
    const max = sortedTimes[sortedTimes.length - 1]
    const median = sortedTimes[Math.floor(sortedTimes.length / 2)]
    
    return {
      name: name,
      iterations: iterations,
      average: avg,
      min: min,
      max: max,
      median: median,
      times: times
    }
  }
  
  static clearMeasurements(): void {
    this.measurements.clear()
  }
}

interface BenchmarkResult {
  name: string
  iterations: number
  average: number
  min: number
  max: number
  median: number
  times: number[]
}

// 使用示例
const benchmarkResult = await PerformanceBenchmark.benchmark(
  'arraySort',
  () => {
    const arr = Array.from({ length: 10000 }, () => Math.random())
    return arr.sort((a, b) => a - b)
  },
  100
)

console.log(`平均耗时: ${benchmarkResult.average}ms`)
console.log(`最小耗时: ${benchmarkResult.min}ms`)
console.log(`最大耗时: ${benchmarkResult.max}ms`)
console.log(`中位数: ${benchmarkResult.median}ms`)
```

### 最佳实践

```typescript
// 5. 测量装饰器
function trace(name?: string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value
    const traceName = name || `${target.constructor.name}.${propertyKey}`
    
    descriptor.value = function (...args: any[]) {
      hiTraceMeter.startTrace(traceName)
      try {
        return originalMethod.apply(this, args)
      } finally {
        hiTraceMeter.finishTrace(traceName, false)
      }
    }
    
    return descriptor
  }
}

// 使用示例
class DataService {
  @trace('DataService.loadData')
  loadData(): Promise<any[]> {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve([{ id: 1, name: 'Item 1' }])
      }, 100)
    })
  }
}

// 6. 性能阈值监控
class PerformanceMonitor {
  private static thresholds: Map<string, number> = new Map()
  
  static setThreshold(name: string, threshold: number): void {
    this.thresholds.set(name, threshold)
  }
  
  static async measureWithThreshold<T>(
    name: string,
    func: () => Promise<T>
  ): Promise<T> {
    const start = Date.now()
    try {
      return await func()
    } finally {
      const duration = Date.now() - start
      const threshold = this.thresholds.get(name)
      
      if (threshold && duration > threshold) {
        console.warn(`性能警告: ${name} 耗时 ${duration}ms，超过阈值 ${threshold}ms`)
      }
    }
  }
}

// 使用示例
PerformanceMonitor.setThreshold('networkRequest', 1000)
await PerformanceMonitor.measureWithThreshold('networkRequest', async () => {
  return await fetch('https://api.example.com/data')
})
```

## HiTrace 函数调用链路追踪

### 基础用法

```typescript
import hiTraceChain from '@ohos.hiTraceChain'

// 1. 开始追踪
const chainId = hiTraceChain.begin('mainOperation', {
  // 可选参数
  // appName: 'MyApp',
  // appVersion: '1.0.0',
  // deviceName: 'MyDevice'
})

// 2. 在函数中创建追踪点
function processData(data: any): any {
  const spanId = hiTraceChain.createSpan('processData')
  // 处理数据...
  const result = data.transform()
  hiTraceChain.endSpan(spanId, false)
  return result
}

// 3. 结束追踪
hiTraceChain.end(chainId, false)
```

### 异步追踪

```typescript
class AsyncTraceManager {
  static async traceAsyncOperation<T>(
    name: string,
    operation: () => Promise<T>
  ): Promise<T> {
    const chainId = hiTraceChain.begin(name)
    
    try {
      return await operation()
    } catch (error) {
      hiTraceChain.end(chainId, true)
      throw error
    } finally {
      hiTraceChain.end(chainId, false)
    }
  }
}

// 使用示例
const result = await AsyncTraceManager.traceAsyncOperation(
  'asyncOperation',
  async () => {
    // 第一步
    await step1()
    
    // 第二步
    await step2()
    
    // 第三步
    return await step3()
  }
)
```

### 跨模块追踪

```typescript
// 模块1: 数据层
class DataRepository {
  async fetchData(id: string): Promise<any> {
    const spanId = hiTraceChain.createSpan('DataRepository.fetchData')
    
    try {
      // 模拟数据库查询
      await new Promise(resolve => setTimeout(resolve, 50))
      
      const data = { id: id, name: 'Sample Data' }
      hiTraceChain.endSpan(spanId, false)
      
      return data
    } catch (error) {
      hiTraceChain.endSpan(spanId, true)
      throw error
    }
  }
}

// 模块2: 业务逻辑层
class BusinessService {
  private dataRepository: DataRepository = new DataRepository()
  
  async processUser(userId: string): Promise<any> {
    const spanId = hiTraceChain.createSpan('BusinessService.processUser')
    
    try {
      // 调用数据层
      const userData = await this.dataRepository.fetchData(userId)
      
      // 业务处理
      const processedData = this.transform(userData)
      
      hiTraceChain.endSpan(spanId, false)
      return processedData
    } catch (error) {
      hiTraceChain.endSpan(spanId, true)
      throw error
    }
  }
  
  private transform(data: any): any {
    const spanId = hiTraceChain.createSpan('BusinessService.transform')
    
    try {
      // 数据转换逻辑
      const result = {
        ...data,
        processed: true
      }
      
      hiTraceChain.endSpan(spanId, false)
      return result
    } catch (error) {
      hiTraceChain.endSpan(spanId, true)
      throw error
    }
  }
}

// 使用示例
const chainId = hiTraceChain.begin('userWorkflow')
try {
  const businessService = new BusinessService()
  const result = await businessService.processUser('user123')
  console.log('处理结果:', result)
} finally {
  hiTraceChain.end(chainId, false)
}
```

## 性能监控与检测

### 运行时性能监控

```typescript
import perf from '@ohos.perf'

class RuntimePerformanceMonitor {
  private static collector: perf.PerfCollector | null = null
  
  static async startMonitoring(): Promise<void> {
    try {
      // 创建性能收集器
      this.collector = await perf.getPerfCollector()
      
      // 配置监控参数
      const config: perf.CollectorConfig = {
        // 监控应用卡顿
        appPids: [], // 空数组表示监控当前应用
        
        // 监控特定线程
        threadNames: [],
        
        // 监控系统资源
        monitorCpu: true,
        monitorMemory: true,
        monitorIo: false,
        monitorNetwork: true,
        
        // 采样间隔（毫秒）
        sampleInterval: 1000,
        
        // 采样数量
        sampleCount: 60
      }
      
      // 启动监控
      await this.collector.start(config)
      
      console.log('性能监控已启动')
    } catch (error) {
      console.error('启动性能监控失败:', error)
      throw error
    }
  }
  
  static async stopMonitoring(): Promise<perf.PerfReport> {
    if (!this.collector) {
      throw new Error('性能监控未启动')
    }
    
    try {
      // 停止监控并获取报告
      const report = await this.collector.stop()
      
      // 分析报告
      this.analyzeReport(report)
      
      return report
    } catch (error) {
      console.error('停止性能监控失败:', error)
      throw error
    }
  }
  
  private static analyzeReport(report: perf.PerfReport): void {
    console.log('=== 性能分析报告 ===')
    
    // 1. CPU使用率分析
    if (report.cpuStats) {
      const avgCpu = report.cpuStats.reduce((sum, stat) => sum + stat.usage, 0) / report.cpuStats.length
      console.log(`平均CPU使用率: ${avgCpu.toFixed(2)}%`)
      
      if (avgCpu > 80) {
        console.warn('警告: CPU使用率过高，建议优化')
      }
    }
    
    // 2. 内存使用分析
    if (report.memoryStats) {
      const avgMemory = report.memoryStats.reduce((sum, stat) => sum + stat.heapSize, 0) / report.memoryStats.length
      const maxMemory = Math.max(...report.memoryStats.map(stat => stat.heapSize))
      
      console.log(`平均内存使用: ${avgMemory}KB`)
      console.log(`峰值内存使用: ${maxMemory}KB`)
      
      if (maxMemory > 500 * 1024) { // 超过500MB
        console.warn('警告: 内存峰值过高，建议优化')
      }
    }
    
    // 3. 网络分析
    if (report.networkStats) {
      const avgLatency = report.networkStats.reduce((sum, stat) => sum + stat.latency, 0) / report.networkStats.length
      const totalBytes = report.networkStats.reduce((sum, stat) => sum + stat.sentBytes + stat.receivedBytes, 0)
      
      console.log(`平均网络延迟: ${avgLatency.toFixed(2)}ms`)
      console.log(`总网络流量: ${(totalBytes / 1024).toFixed(2)}KB`)
    }
    
    // 4. 异常分析
    if (report.exceptions && report.exceptions.length > 0) {
      console.error(`检测到 ${report.exceptions.length} 个异常:`)
      report.exceptions.forEach((exception, index) => {
        console.error(`${index + 1}. ${exception.type}: ${exception.message}`)
        console.error(`   时间: ${new Date(exception.timestamp).toISOString()}`)
        console.error(`   堆栈: ${exception.stackTrace}`)
      })
    }
  }
}

// 使用示例
// 启动监控
await RuntimePerformanceMonitor.startMonitoring()

// 执行一些操作
await someAsyncOperation()

// 停止监控并获取报告
const report = await RuntimePerformanceMonitor.stopMonitoring()
```

### 内存泄漏检测

```typescript
import perf from '@ohos.perf'

class MemoryLeakDetector {
  private static snapshots: perf.MemorySnapshot[] = []
  
  static async takeSnapshot(): Promise<perf.MemorySnapshot> {
    try {
      const collector = await perf.getPerfCollector()
      const snapshot = await collector.captureHeapSnapshot()
      
      this.snapshots.push(snapshot)
      
      console.log(`已捕获内存快照，当前快照数量: ${this.snapshots.length}`)
      
      return snapshot
    } catch (error) {
      console.error('捕获内存快照失败:', error)
      throw error
    }
  }
  
  static async compareSnapshots(
    snapshot1: perf.MemorySnapshot,
    snapshot2: perf.MemorySnapshot
  ): Promise<MemoryDiffResult> {
    try {
      // 比较两个快照的差异
      const diff = await perf.compareHeapSnapshots(snapshot1, snapshot2)
      
      // 分析内存增长
      const sizeDelta = diff.sizeDelta
      const objectCountDelta = diff.objectCountDelta
      
      console.log(`内存大小差异: ${sizeDelta}B`)
      console.log(`对象数量差异: ${objectCountDelta}`)
      
      // 检测泄漏
      if (sizeDelta > 10 * 1024 * 1024) { // 增长超过10MB
        console.warn('警告: 检测到内存泄漏')
        console.log('新增对象数量:', diff.newObjects.length)
        
        // 分析泄漏对象
        diff.newObjects.slice(0, 10).forEach((obj, index) => {
          console.log(`${index + 1}. ${obj.type}: ${obj.size}B`)
          if (obj.references.length > 0) {
            console.log(`   引用数: ${obj.references.length}`)
          }
        })
      }
      
      return {
        sizeDelta: sizeDelta,
        objectCountDelta: objectCountDelta,
        newObjects: diff.newObjects,
        leakedObjects: diff.leakedObjects,
        retentionPath: diff.retentionPath
      }
    } catch (error) {
      console.error('比较内存快照失败:', error)
      throw error
    }
  }
}

interface MemoryDiffResult {
  sizeDelta: number
  objectCountDelta: number
  newObjects: perf.MemoryObject[]
  leakedObjects: perf.MemoryObject[]
  retentionPath: perf.ReferencePath
}

// 使用示例
async function detectMemoryLeak() {
  const detector = new MemoryLeakDetector()
  
  // 1. 捕获初始快照
  const snapshot1 = await detector.takeSnapshot()
  
  // 2. 执行可能泄漏的操作
  await performPotentiallyLeakyOperation()
  
  // 3. 捕获操作后快照
  const snapshot2 = await detector.takeSnapshot()
  
  // 4. 比较快照
  const diff = await detector.compareSnapshots(snapshot1, snapshot2)
  
  if (diff.sizeDelta > 10 * 1024 * 1024) {
    console.error('检测到严重内存泄漏!')
    console.log('建议检查以下对象的引用关系:')
    diff.leakedObjects.slice(0, 5).forEach((obj, index) => {
      console.log(`${index + 1}. ${obj.type} (${obj.size}B)`)
    })
  }
}
```

### 应用冻结（ANR）检测

```typescript
import perf from '@ohos.perf'

class AppFreezeMonitor {
  private static monitor: perf.AppFreezeMonitor | null = null
  
  static async startMonitoring(): Promise<void> {
    try {
      this.monitor = await perf.getAppFreezeMonitor()
      
      // 配置监控
      const config: perf.FreezeConfig = {
        // 冻结超时阈值（毫秒）
        freezeThreshold: 5000,
        
        // 监控模式
        mode: perf.FreezeMode.AUTOMATIC,
        
        // 是否启用系统冻结检测
        enableSystemFreeze: true,
        
        // 是否启用应用冻结检测
        enableAppFreeze: true,
        
        // 回调函数
        onFreeze: (freezeInfo: perf.FreezeInfo) => {
          console.error('检测到应用冻结:', freezeInfo)
          console.error(`冻结时间: ${freezeInfo.duration}ms`)
          console.error(`冻结位置: ${freezeInfo.stackTrace}`)
          
          // 上报冻结信息
          this.reportFreeze(freezeInfo)
        },
        
        // 阈值回调
        onThresholdExceeded: (thresholdInfo: perf.ThresholdInfo) => {
          console.warn('超过冻结阈值:', thresholdInfo)
        }
      }
      
      await this.monitor.start(config)
      console.log('应用冻结监控已启动')
    } catch (error) {
      console.error('启动应用冻结监控失败:', error)
      throw error
    }
  }
  
  static async stopMonitoring(): Promise<void> {
    if (!this.monitor) {
      throw new Error('应用冻结监控未启动')
    }
    
    await this.monitor.stop()
    console.log('应用冻结监控已停止')
  }
  
  private static reportFreeze(freezeInfo: perf.FreezeInfo): void {
    // 上报冻结信息到服务器
    const freezeReport = {
      timestamp: Date.now(),
      duration: freezeInfo.duration,
      stackTrace: freezeInfo.stackTrace,
      deviceInfo: this.getDeviceInfo(),
      appInfo: this.getAppInfo()
    }
    
    // 这里应该调用上报API
    console.log('上报冻结信息:', freezeReport)
  }
  
  private static getDeviceInfo(): any {
    // 获取设备信息
    return {
      model: 'Unknown',
      osVersion: 'Unknown'
    }
  }
  
  private static getAppInfo(): any {
    // 获取应用信息
    return {
      packageName: 'com.example.app',
      version: '1.0.0',
      versionCode: 1
    }
  }
}

// 使用示例
await AppFreezeMonitor.startMonitoring()

// 在应用运行期间...

await AppFreezeMonitor.stopMonitoring()
```

## 性能优化策略

### CPU优化

```typescript
// 1. 减少主线程工作量
class CpuOptimizer {
  // 使用Worker处理耗时任务
  static async processInWorker<T>(
    task: () => Promise<T>,
    callback: (result: T) => void
  ): Promise<void> {
    // 创建Worker
    const worker = new Worker('workers/heavy-task.js')
    
    // 发送任务
    worker.postMessage({ task: 'heavyOperation' })
    
    // 监听结果
    worker.onmessage = (event) => {
      callback(event.data.result)
      worker.terminate()
    }
    
    worker.onerror = (error) => {
      console.error('Worker错误:', error)
      worker.terminate()
    }
  }
  
  // 使用TaskPool进行并发处理
  static async parallelProcess<T, R>(
    items: T[],
    processor: (item: T) => Promise<R>,
    concurrency: number = 4
  ): Promise<R[]> {
    const taskPool = new TaskPool(concurrency)
    const results: R[] = []
    
    const tasks = items.map(item => 
      taskPool.execute(() => processor(item))
    )
    
    await Promise.all(tasks)
    return results
  }
}

// 使用示例
// Worker模式
await CpuOptimizer.processInWorker(
  () => new Promise(resolve => {
    setTimeout(() => {
      resolve({ result: 'Processed in worker' })
    }, 1000)
  }),
  (result) => {
    console.log('处理结果:', result)
  }
)

// TaskPool模式
const items = Array.from({ length: 100 }, (_, i) => i)
const results = await CpuOptimizer.parallelProcess(
  items,
  async (item) => {
    await new Promise(resolve => setTimeout(resolve, 10))
    return item * 2
  },
  8 // 8个并发任务
)
```

### 内存优化

```typescript
class MemoryOptimizer {
  // 1. 对象池模式
  private static pools: Map<string, any[]> = new Map()
  
  static getFromPool<T>(type: string, factory: () => T): T {
    let pool = this.pools.get(type)
    
    if (!pool || pool.length === 0) {
      return factory()
    }
    
    return pool.pop()
  }
  
  static returnToPool<T>(type: string, obj: T): void {
    let pool = this.pools.get(type)
    
    if (!pool) {
      pool = []
      this.pools.set(type, pool)
    }
    
    pool.push(obj)
  }
  
  // 使用示例
  interface DataItem {
    id: number
    data: string
  }
  
  const item = MemoryOptimizer.getFromPool<DataItem>('DataItem', () => ({
    id: Date.now(),
    data: 'Sample'
  }))
  
  // 使用item...
  
  MemoryOptimizer.returnToPool('DataItem', item)
  
  // 2. 大对象分块处理
  static async processLargeData<T>(
    data: T[],
    chunkSize: number,
    processor: (chunk: T[]) => Promise<void>
  ): Promise<void> {
    for (let i = 0; i < data.length; i += chunkSize) {
      const chunk = data.slice(i, i + chunkSize)
      await processor(chunk)
      
      // 给垃圾回收时间
      await new Promise(resolve => setTimeout(resolve, 10))
    }
  }
  
  // 3. 弱引用避免内存泄漏
  static createWeakMap<K extends object, V>(): WeakMap<K, V> {
    return new WeakMap<K, V>()
  }
  
  static createWeakRef<T>(obj: T): WeakRef<T> {
    return new WeakRef(obj)
  }
}

// 使用示例
// 对象池
interface ListItem {
  id: number
  content: string
}

const listItem = MemoryOptimizer.getFromPool<ListItem>('ListItem', () => ({
  id: 0,
  content: ''
}))

// 使用listItem...
listItem.content = 'Updated content'

// 回收到池
MemoryOptimizer.returnToPool('ListItem', listItem)

// 弱引用
interface UserData {
  id: string
  name: string
}

const userMap = MemoryOptimizer.createWeakMap<object, UserData>()

// 使用WeakRef
const componentRef = new WeakRef(myComponent)

// 后续使用
const component = componentRef.deref()
if (component) {
  // 组件仍然存活
  component.doSomething()
}
```

### 网络优化

```typescript
class NetworkOptimizer {
  // 1. 请求合并
  private static pendingRequests: Map<string, Promise<any>[]> = new Map()
  
  static async mergeRequest<T>(
    key: string,
    request: () => Promise<T>,
    maxWaitTime: number = 100
  ): Promise<T> {
    let requests = this.pendingRequests.get(key)
    
    if (!requests) {
      requests = []
      this.pendingRequests.set(key, requests)
      
      // 延迟执行合并
      setTimeout(async () => {
        try {
          const result = await request()
          
          // 通知所有等待的请求
          requests.forEach(promise => {
            promise.resolve(result)
          })
          
          this.pendingRequests.delete(key)
        } catch (error) {
          // 通知所有等待的请求
          requests.forEach(promise => {
            promise.reject(error)
          })
          
          this.pendingRequests.delete(key)
        }
      }, maxWaitTime)
    }
    
    return new Promise((resolve, reject) => {
      requests.push({ resolve, reject })
    })
  }
  
  // 2. 请求缓存
  private static cache: Map<string, CacheEntry> = new Map()
  
  static async fetchWithCache<T>(
    url: string,
    fetchFn: () => Promise<T>,
    ttl: number = 5 * 60 * 1000 // 5分钟缓存
  ): Promise<T> {
    const now = Date.now()
    const cacheKey = url
    
    // 检查缓存
    const cached = this.cache.get(cacheKey)
    if (cached && now - cached.timestamp < ttl) {
      console.log(`使用缓存数据: ${cacheKey}`)
      return cached.data
    }
    
    // 发起请求
    const data = await fetchFn()
    
    // 缓存结果
    this.cache.set(cacheKey, {
      data: data,
      timestamp: now
    })
    
    return data
  }
  
  // 3. 请求队列
  private static requestQueue: Map<string, Promise<void>> = new Map()
  
  static async queueRequest(
    key: string,
    request: () => Promise<void>
  ): Promise<void> {
    const existing = this.requestQueue.get(key)
        
    if (existing) {
      return existing
    }
    
    const promise = request().finally(() => {
      this.requestQueue.delete(key)
    })
    
    this.requestQueue.set(key, promise)
    
    return promise
  }
}

interface CacheEntry {
  data: any
  timestamp: number
}

// 使用示例
// 请求合并
const userData1 = await NetworkOptimizer.mergeRequest(
  'userData',
  () => fetch('https://api.example.com/user'),
  100
)

const userData2 = await NetworkOptimizer.mergeRequest(
  'userData',
  () => fetch('https://api.example.com/user'),
  100
)

// 两个请求会被合并为一个

// 请求缓存
const data1 = await NetworkOptimizer.fetchWithCache(
  'https://api.example.com/data',
  () => fetch('https://api.example.com/data').then(r => r.json()),
  10 * 60 * 1000 // 10分钟缓存
)

const data2 = await NetworkOptimizer.fetchWithCache(
  'https://api.example.com/data',
  () => fetch('https://api.example.com/data').then(r => r.json()),
  10 * 60 * 1000
)

// 第二次请求会使用缓存

// 请求队列
await NetworkOptimizer.queueRequest('upload', async () => {
  await uploadFile(file1)
})

await NetworkOptimizer.queueRequest('upload', async () => {
  await uploadFile(file2)
})

// 第二个上传会等待第一个完成后执行
```

## 调试技巧与工具集成

### 集成DevEco Studio

```typescript
// 1. 使用@OH_Trace进行性能追踪
import { OH_Trace } from '@ohos.hiTraceChain'

class TracedComponent {
  @OH_Trace
  performExpensiveOperation(): void {
    // 自动追踪此函数的性能
    let result = 0
    for (let i = 0; i < 10000; i++) {
      result += i
    }
  }
}

// 2. 使用@OH_Perf进行性能标记
import { OH_Perf } from '@ohos.hiTraceMeter'

class PerfComponent {
  @OH_Perf
  optimizedFunction(): void {
    // 自动测量此函数的执行时间
    this.doWork()
  }
  
  private doWork(): void {
    // 业务逻辑
  }
}

// 3. 条件追踪
class ConditionalTracer {
  static traceIf(
    condition: boolean,
    name: string,
    fn: () => void
  ): void {
    if (condition) {
      hiTraceMeter.startTrace(name)
      try {
        fn()
      } finally {
        hiTraceMeter.finishTrace(name, false)
      }
    } else {
      fn()
    }
  }
}

// 使用示例
const isDebugMode = true

ConditionalTracer.traceIf(
  isDebugMode,
  'debugOperation',
  () => {
    // 只在调试模式下追踪
    console.log('执行调试操作')
  }
)
```

### 性能分析工作流

```typescript
// 性能分析工作流
class PerformanceAnalysisWorkflow {
  static async analyzePerformance(): Promise<PerformanceAnalysisReport> {
    const report: PerformanceAnalysisReport = {
      timestamp: Date.now(),
      phases: [],
      issues: [],
      recommendations: []
    }
    
    // 阶段1: 启动性能分析
    console.log('阶段1: 启动性能分析')
    const startupPhase = await this.analyzeStartup()
    report.phases.push(startupPhase)
    
    // 阶段2: 运行时性能分析
    console.log('阶段2: 运行时性能分析')
    const runtimePhase = await this.analyzeRuntime()
    report.phases.push(runtimePhase)
    
    // 阶段3: 内存分析
    console.log('阶段3: 内存分析')
    const memoryPhase = await this.analyzeMemory()
    report.phases.push(memoryPhase)
    
    // 阶段4: 生成建议
    console.log('阶段4: 生成优化建议')
    report.recommendations = this.generateRecommendations(report.phases)
    
    // 汇总问题
    report.issues = this.collectIssues(report.phases)
    
    return report
  }
  
  private static async analyzeStartup(): Promise<AnalysisPhase> {
    // 分析启动性能
    return {
      name: '启动性能分析',
      metrics: {
        coldStartTime: 2500,
        warmStartTime: 800,
        firstFrameTime: 3200
      },
      status: 'pass' // pass, warning, fail
    }
  }
  
  private static async analyzeRuntime(): Promise<AnalysisPhase> {
    // 分析运行时性能
    return {
      name: '运行时性能分析',
      metrics: {
        averageFps: 58,
        frameDropRate: 2.5,
        memoryUsage: 150 * 1024 * 1024
      },
      status: 'warning'
    }
  }
  
  private static async analyzeMemory(): Promise<AnalysisPhase> {
    // 分析内存使用
    return {
      name: '内存分析',
      metrics: {
        peakMemory: 200 * 1024 * 1024,
        averageMemory: 120 * 1024 * 1024,
        memoryLeaks: 0
      },
      status: 'pass'
    }
  }
  
  private static generateRecommendations(phases: AnalysisPhase[]): string[] {
    const recommendations: string[] = []
    
    phases.forEach(phase => {
      if (phase.status === 'fail') {
        recommendations.push(`${phase.name}: 需要立即优化`)
      } else if (phase.status === 'warning') {
        recommendations.push(`${phase.name}: 建议优化`)
      }
    })
    
    return recommendations
  }
  
  private static collectIssues(phases: AnalysisPhase[]): Issue[] {
    const issues: Issue[] = []
    
    phases.forEach(phase => {
      if (phase.status !== 'pass') {
        issues.push({
          phase: phase.name,
          severity: phase.status,
          metrics: phase.metrics
        })
      }
    })
    
    return issues
  }
}

interface AnalysisPhase {
  name: string
  metrics: any
  status: 'pass' | 'warning' | 'fail'
}

interface PerformanceAnalysisReport {
  timestamp: number
  phases: AnalysisPhase[]
  issues: Issue[]
  recommendations: string[]
}

interface Issue {
  phase: string
  severity: 'pass' | 'warning' | 'fail'
  metrics: any
}

// 使用示例
const report = await PerformanceAnalysisWorkflow.analyzePerformance()

console.log('性能分析报告:')
console.log('问题数量:', report.issues.length)
console.log('建议数量:', report.recommendations.length)

report.recommendations.forEach(rec => {
  console.log('-', rec)
})
```

## 最佳实践总结

### 1. 性能分析时机

- **开发阶段**：定期进行性能分析，及时发现性能问题
- **测试阶段**：在各种设备和网络环境下进行性能测试
- **上线前**：进行全面的性能分析和优化
- **上线后**：持续监控线上应用的性能指标

### 2. 性能指标监控

| 指标 | 目标值 | 告警阈值 | 严重阈值 |
|------|--------|---------|---------|
| 启动时间 | < 3s | 3-5s | > 5s |
| 帧率 (FPS) | ≥ 60 | 45-59 | < 45 |
| 内存使用 | < 150MB | 150-200MB | > 200MB |
| CPU使用率 | < 50% | 50-80% | > 80% |
| 网络延迟 | < 100ms | 100-300ms | > 300ms |

### 3. 代码优化原则

- **避免不必要的对象创建**：复用对象，使用对象池
- **减少内存分配**：使用基本类型，避免装箱拆箱
- **优化算法复杂度**：选择合适的算法和数据结构
- **异步处理耗时操作**：使用Worker、TaskPool等并发机制
- **合理使用缓存**：减少重复计算和网络请求
- **及时释放资源**：在合适的时机释放不再使用的资源

### 4. 调试技巧

- **使用性能分析工具**：DevEco Profiler、HiTrace、HiTraceMeter
- **记录详细的日志**：便于定位问题
- **复现问题**：提供稳定的复现步骤
- **分步调试**：逐步缩小问题范围
- **对比分析**：对比正常和异常情况下的差异

## 参考

- [华为官方文档 - HiTraceMeter性能测量](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hitracemeter-guidelines-arkts)
- [华为官方文档 - HiTrace链路追踪](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hitracechain-guidelines-arkts)
- [华为官方文档 - 性能检测](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/perf-detection)
- [华为官方文档 - 应用冻结检测](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/appfreeze-guidelines)
- [华为官方文档 - 资源泄漏检测](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/resource-leak-guidelines)
- [华为官方文档 - 功耗检测](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/power-detection)