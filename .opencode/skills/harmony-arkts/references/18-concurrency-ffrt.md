# HarmonyOS FFRT 并发框架与任务调度指南

## FFRT (Function Reaction to Task) 概述

FFRT 是 HarmonyOS 提供的高性能并发任务调度框架，专为异构多核处理器设计，提供以下核心能力：

| 能力 | 说明 | 适用场景 |
|------|------|---------|
| **任务优先级** | 支持多级优先级调度 | 关键任务优先执行 |
| **任务依赖** | 支持任务间的依赖关系 | 复杂工作流编排 |
| **任务组** | 支持任务组管理和统一控制 | 批量任务管理 |
| **负载均衡** | 智能任务分配到不同核心 | CPU密集型任务优化 |
| **任务取消** | 支持任务取消和超时控制 | 资源释放和异常处理 |

## 基础任务创建与执行

### 同步任务

```typescript
import ffrt from '@ohos.worker.frt'

// 1. 创建同步任务
function createSyncTask(name: string): void {
  const task: ffrt.Task = {
    name: name,
    handler: () => {
      console.log(`执行任务: ${name}`)
      // 任务逻辑
      for (let i = 0; i < 1000000; i++) {
        // 模拟耗时操作
      }
      console.log(`任务 ${name} 完成`)
    }
  }
  
  // 提交任务
  ffrt.submit(task)
}

// 使用示例
createSyncTask('syncTask1')
createSyncTask('syncTask2')
```

### 异步任务

```typescript
// 2. 创建异步任务
function createAsyncTask(name: string, delay: number): void {
  const task: ffrt.Task = {
    name: name,
    handler: async () => {
      console.log(`开始异步任务: ${name}`)
      await new Promise(resolve => setTimeout(resolve, delay))
      console.log(`异步任务 ${name} 完成`)
    }
  }
  
  // 提交任务
  ffrt.submit(task)
}

// 使用示例
createAsyncTask('asyncTask1', 1000)
createAsyncTask('asyncTask2', 2000)
```

### 带返回值的任务

```typescript
// 3. 带返回值的任务
function createTaskWithResult(): void {
  const task: ffrt.Task = {
    name: 'taskWithResult',
    handler: () => {
      // 计算结果
      const result = {
        success: true,
        data: Math.random() * 100,
        timestamp: Date.now()
      }
      console.log(`任务结果: ${JSON.stringify(result)}`)
      return result
    }
  }
  
  // 提交任务并获取结果
  ffrt.submitWithResult(task)
}

// 使用示例
createTaskWithResult()
```

## 任务优先级与调度

### 优先级设置

```typescript
// 4. 设置任务优先级
enum TaskPriority {
  HIGH = 0,      // 高优先级
  MEDIUM = 1,    // 中优先级
  LOW = 2        // 低优先级
}

function createPriorityTask(name: string, priority: TaskPriority): void {
  const task: ffrt.Task = {
    name: name,
    priority: priority,
    handler: () => {
      console.log(`执行${priority === TaskPriority.HIGH ? '高' : priority === TaskPriority.MEDIUM ? '中' : '低'}优先级任务: ${name}`)
      // 任务逻辑
    }
  }
  
  // 提交任务
  ffrt.submit(task)
}

// 使用示例
createPriorityTask('urgentTask', TaskPriority.HIGH)
createPriorityTask('normalTask', TaskPriority.MEDIUM)
createPriorityTask('backgroundTask', TaskPriority.LOW)
```

### 动态优先级调整

```typescript
// 5. 动态调整任务优先级
class PriorityTaskManager {
  private static tasks: Map<string, ffrt.Task> = new Map()
  
  static createTask(name: string, initialPriority: TaskPriority): void {
    const task: ffrt.Task = {
      name: name,
      priority: initialPriority,
      handler: () => {
        console.log(`执行任务: ${name}, 优先级: ${task.priority}`)
        // 任务执行完成后更新优先级
        this.updateTaskPriority(name, TaskPriority.LOW)
      }
    }
    
    this.tasks.set(name, task)
    ffrt.submit(task)
  }
  
  static updateTaskPriority(taskName: string, newPriority: TaskPriority): void {
    const task = this.tasks.get(taskName)
    if (task) {
      task.priority = newPriority
      console.log(`更新任务 ${taskName} 优先级为 ${newPriority}`)
      // 重新提交任务以应用新优先级
      ffrt.submit(task)
    }
  }
}

// 使用示例
PriorityTaskManager.createTask('dataSync', TaskPriority.HIGH)

// 任务执行后降低优先级
PriorityTaskManager.updateTaskPriority('dataSync', TaskPriority.LOW)
```

## 任务组管理

### 创建任务组

```typescript
// 6. 任务组管理
class TaskGroup {
  private groupName: string
  private tasks: ffrt.Task[] = []
  private completionCount: number = 0
  private completionCallback?: () => void
  
  constructor(name: string, onComplete?: () => void) {
    this.groupName = name
    this.completionCallback = onComplete
  }
  
  addTask(task: ffrt.Task): void {
    this.tasks.push(task)
    console.log(`任务组 ${this.groupName} 添加任务: ${task.name}`)
  }
  
  async execute(): Promise<void> {
    console.log(`开始执行任务组: ${this.groupName}, 任务数量: ${this.tasks.length}`)
    
    const promises = this.tasks.map(task => {
      return new Promise<void>((resolve) => {
        const wrappedTask: ffrt.Task = {
          name: task.name,
          priority: task.priority,
          handler: () => {
            task.handler()
            this.completionCount++
            console.log(`任务组 ${this.groupName} 完成 ${this.completionCount}/${this.tasks.length}`)
            
            if (this.completionCount === this.tasks.length) {
              console.log(`任务组 ${this.groupName} 全部完成`)
              if (this.completionCallback) {
                this.completionCallback()
              }
            }
            
            resolve()
          }
        }
        
        ffrt.submit(wrappedTask)
      })
    })
    
    await Promise.all(promises)
  }
}

// 使用示例
const downloadGroup = new TaskGroup('downloadGroup', () => {
  console.log('下载任务组全部完成')
})

// 添加下载任务
downloadGroup.addTask({
  name: 'downloadImage1',
  priority: TaskPriority.HIGH,
  handler: () => {
    console.log('下载图片1')
  }
})

downloadGroup.addTask({
  name: 'downloadImage2',
  priority: TaskPriority.MEDIUM,
  handler: () => {
    console.log('下载图片2')
  }
})

downloadGroup.addTask({
  name: 'downloadImage3',
  priority: TaskPriority.LOW,
  handler: () => {
    console.log('下载图片3')
  }
})

// 执行任务组
await downloadGroup.execute()
```

## 任务依赖与工作流

### 串行队列

```typescript
// 7. 串行队列
class SerialTaskQueue {
  private queue: (() => Promise<void>)[] = []
  private isProcessing: boolean = false
  
  async enqueue(task: () => Promise<void>): Promise<void> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          await task()
          resolve()
        } catch (error) {
          reject(error)
        }
      })
      
      this.process()
    })
  }
  
  private async process(): Promise<void> {
    if (this.isProcessing || this.queue.length === 0) {
      return
    }
    
    this.isProcessing = true
    
    try {
      const task = this.queue.shift()!
      await task()
    } catch (error) {
      console.error('任务执行失败:', error)
    } finally {
      this.isProcessing = false
      // 处理下一个任务
      this.process()
    }
  }
}

// 使用示例
const serialQueue = new SerialTaskQueue()

await serialQueue.enqueue(async () => {
  console.log('任务1')
  await new Promise(resolve => setTimeout(resolve, 1000))
})

await serialQueue.enqueue(async () => {
  console.log('任务2')
  await new Promise(resolve => setTimeout(resolve, 1000))
})

await serialQueue.enqueue(async () => {
  console.log('任务3')
  await new Promise(resolve => setTimeout(resolve, 1000))
})
```

### 并发队列

```typescript
// 8. 并发队列
class ConcurrentTaskQueue {
  private queue: (() => Promise<void>)[] = []
  private maxConcurrency: number
  private activeCount: number = 0
  
  constructor(maxConcurrency: number = 4) {
    this.maxConcurrency = maxConcurrency
  }
  
  async enqueue(task: () => Promise<void>): Promise<void> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          await task()
          resolve()
        } catch (error) {
          reject(error)
        }
      })
      
      this.process()
    })
  }
  
  private async process(): Promise<void> {
    while (this.activeCount < this.maxConcurrency && this.queue.length > 0) {
      this.activeCount++
      const task = this.queue.shift()!
      
      task().finally(() => {
        this.activeCount--
        this.process()
      })
    }
  }
}

// 使用示例
const concurrentQueue = new ConcurrentTaskQueue(3) // 最大并发3个

for (let i = 0; i < 10; i++) {
  concurrentQueue.enqueue(async () => {
    console.log(`开始任务 ${i}`)
    await new Promise(resolve => setTimeout(resolve, 1000))
    console.log(`完成任务 ${i}`)
  })
}
```

### 并行图

```typescript
// 9. 并行图
import graph from '@ohos.graph'

class ParallelGraphExecutor {
  private graph: graph.Graph
  private executor: ffrt.Executor
  
  constructor() {
    this.graph = graph.createGraph()
    this.executor = new ffrt.Executor()
  }
  
  addTask(taskId: string, dependencies: string[], handler: () => void): void {
    const task: ffrt.Task = {
      id: taskId,
      name: taskId,
      dependencies: dependencies,
      handler: handler
    }
    
    this.graph.addTask(task)
  }
  
  async execute(): Promise<void> {
    const sortedTasks = this.graph.topologicalSort()
    
    console.log(`任务执行顺序: ${sortedTasks.map(t => t.id).join(' -> ')}`)
    
    const promises = sortedTasks.map(task => {
      return new Promise<void>((resolve) => {
        const wrappedTask: ffrt.Task = {
          name: task.name,
          handler: () => {
            task.handler()
            resolve()
          }
        }
        
        ffrt.submit(wrappedTask)
      })
    })
    
    await Promise.all(promises)
  }
}

// 使用示例
const graphExecutor = new ParallelGraphExecutor()

// 添加任务并设置依赖关系
graphExecutor.addTask('taskA', [], () => {
  console.log('执行任务A')
})

graphExecutor.addTask('taskB', ['taskA'], () => {
  console.log('执行任务B (依赖任务A)')
})

graphExecutor.addTask('taskC', ['taskA'], () => {
  console.log('执行任务C (依赖任务A)')
})

graphExecutor.addTask('taskD', ['taskB', 'taskC'], () => {
  console.log('执行任务D (依赖任务B和任务C)')
})

await graphExecutor.execute()
```

## TaskPool 高级应用

### 任务池优化

```typescript
// 10. TaskPool 高级应用
import taskpool from '@ohos.taskpool'

class AdvancedTaskPool {
  private static pool: taskpool.TaskPool
  private static taskQueue: Map<string, taskpool.TaskInfo> = new Map()
  
  static async initialize(): Promise<void> {
    // 创建任务池
    this.pool = new taskpool.TaskPool({
      name: 'AdvancedTaskPool',
      parallelism: 4, // 并行数
      priority: taskpool.Priority.HIGH
    })
    
    console.log('高级任务池已初始化')
  }
  
  static async executeTask<T>(
    taskId: string,
    task: () => Promise<T>,
    priority: taskpool.Priority = taskpool.Priority.MEDIUM
  ): Promise<T> {
    const taskInfo: taskpool.TaskInfo = {
      id: taskId,
      name: taskId,
      priority: priority,
      task: task
    }
    
    this.taskQueue.set(taskId, taskInfo)
    
    try {
      const result = await this.pool.execute(taskInfo)
      return result
    } finally {
      this.taskQueue.delete(taskId)
    }
  }
  
  static async cancelTask(taskId: string): Promise<boolean> {
    const taskInfo = this.taskQueue.get(taskId)
    if (!taskInfo) {
      return false
    }
    
    return await this.pool.cancel(taskInfo)
  }
  
  static async pausePool(): Promise<void> {
    await this.pool.pause()
    console.log('任务池已暂停')
  }
  
  static async resumePool(): Promise<void> {
    await this.pool.resume()
    console.log('任务池已恢复')
  }
}

// 使用示例
await AdvancedTaskPool.initialize()

// 添加高优先级任务
const result1 = await AdvancedTaskPool.executeTask(
  'highPriorityTask',
  async () => {
    console.log('执行高优先级任务')
    await new Promise(resolve => setTimeout(resolve, 1000))
    return 'Result 1'
  },
  taskpool.Priority.HIGH
)

// 添加普通优先级任务
const result2 = await AdvancedTaskPool.executeTask(
  'normalTask',
  async () => {
    console.log('执行普通任务')
    await new Promise(resolve => setTimeout(resolve, 2000))
    return 'Result 2'
  }
)

// 取消任务
const cancelled = await AdvancedTaskPool.cancelTask('normalTask')
console.log('任务取消:', cancelled)
```

### 任务超时控制

```typescript
// 11. 任务超时控制
class TimeoutTask {
  private static timeoutMap: Map<string, NodeJS.Timeout> = new Map()
  
  static executeWithTimeout<T>(
    taskId: string,
    task: () => Promise<T>,
    timeout: number,
    onTimeout?: () => void
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      // 创建任务
      const ffrtTask: ffrt.Task = {
        name: taskId,
        handler: async () => {
          try {
            const result = await task()
            
            // 清除超时计时器
            const timeoutTimer = this.timeoutMap.get(taskId)
            if (timeoutTimer) {
              clearTimeout(timeoutTimer)
              this.timeoutMap.delete(taskId)
            }
            
            resolve(result)
          } catch (error) {
            // 清除超时计时器
            const timeoutTimer = this.timeoutMap.get(taskId)
            if (timeoutTimer) {
              clearTimeout(timeoutTimer)
              this.timeoutMap.delete(taskId)
            }
            
            reject(error)
          }
        }
      }
      
      // 设置超时计时器
      const timeoutTimer = setTimeout(() => {
        console.error(`任务 ${taskId} 超时 (${timeout}ms)`)
        
        if (onTimeout) {
          onTimeout()
        }
        
        // 尝试取消任务
        ffrt.cancel(taskId)
        
        // 清除超时计时器
        this.timeoutMap.delete(taskId)
      }, timeout)
      
      this.timeoutMap.set(taskId, timeoutTimer)
      
      // 提交任务
      ffrt.submit(ffrtTask)
    })
  }
  
  static cancelTimeout(taskId: string): void {
    const timeoutTimer = this.timeoutMap.get(taskId)
    if (timeoutTimer) {
      clearTimeout(timeoutTimer)
      this.timeoutMap.delete(taskId)
      console.log(`已取消任务 ${taskId} 的超时计时器`)
    }
  }
}

// 使用示例
const result = await TimeoutTask.executeWithTimeout(
  'networkRequest',
  async () => {
    console.log('执行网络请求')
    const response = await fetch('https://api.example.com/data')
    return response.json()
  },
  5000, // 5秒超时
  () => {
    console.log('网络请求超时，使用缓存数据')
  }
)
```

## 并发模式与最佳实践

### 生产者-消费者模式

```typescript
// 12. 生产者-消费者模式
class ProducerConsumer {
  private static buffer: any[] = []
  private static maxBufferSize: number = 100
  private static consumers: ffrt.Task[] = []
  
  static async startConsumers(consumerCount: number): Promise<void> {
    for (let i = 0; i < consumerCount; i++) {
      const consumer: ffrt.Task = {
        name: `consumer${i}`,
        priority: TaskPriority.MEDIUM,
        handler: async () => {
          while (true) {
            if (this.buffer.length === 0) {
              // 等待一段时间再检查
              await new Promise(resolve => setTimeout(resolve, 100))
              continue
            }
            
            const item = this.buffer.shift()
            await this.processItem(item)
          }
        }
      }
      
      ffrt.submit(consumer)
      this.consumers.push(consumer)
    }
  }
  
  static async produce(item: any): Promise<void> {
    while (this.buffer.length >= this.maxBufferSize) {
      console.warn('缓冲区已满，等待消费者处理')
      await new Promise(resolve => setTimeout(resolve, 100))
    }
    
    this.buffer.push(item)
    console.log(`生产项目: ${JSON.stringify(item)}, 缓冲区大小: ${this.buffer.length}`)
  }
  
  private static async processItem(item: any): Promise<void> {
    console.log(`消费项目: ${JSON.stringify(item)}`)
    // 模拟处理
    await new Promise(resolve => setTimeout(resolve, 500))
  }
}

// 使用示例
await ProducerConsumer.startConsumers(3)

// 生产者生产项目
for (let i = 0; i < 20; i++) {
  await ProducerConsumer.produce({ id: i, data: `Item ${i}` })
}
```

### 批处理模式

```typescript
// 13. 批处理模式
class BatchProcessor {
  private static batchSize: number = 10
  private static processingInterval: number = 1000
  private static pendingItems: any[] = []
  private static batchTimer: NodeJS.Timeout | null = null
  
  static addItem(item: any): void {
    this.pendingItems.push(item)
    
    if (this.pendingItems.length >= this.batchSize) {
      this.processBatch()
    }
  }
  
  static startBatchProcessor(): void {
    this.batchTimer = setInterval(() => {
      if (this.pendingItems.length > 0) {
        this.processBatch()
      }
    }, this.processingInterval)
  }
  
  static stopBatchProcessor(): void {
    if (this.batchTimer) {
      clearInterval(this.batchTimer)
      this.batchTimer = null
    }
  }
  
  private static processBatch(): void {
    if (this.pendingItems.length === 0) {
      return
    }
    
    const batch = this.pendingItems.splice(0, this.batchSize)
    console.log(`处理批次: ${batch.length} 个项目`)
    
    // 提交批处理任务
    const batchTask: ffrt.Task = {
      name: 'batchProcess',
      priority: TaskPriority.HIGH,
      handler: () => {
        this.processItems(batch)
      }
    }
    
    ffrt.submit(batchTask)
  }
  
  private static async processItems(items: any[]): Promise<void> {
    console.log(`处理 ${items.length} 个项目`)
    
    // 模拟批处理
    await new Promise(resolve => setTimeout(resolve, 2000))
    
    console.log(`批次处理完成`)
  }
}

// 使用示例
BatchProcessor.startBatchProcessor()

// 添加项目
for (let i = 0; i < 35; i++) {
  BatchProcessor.addItem({ id: i, data: `Item ${i}` })
}

// 等待批处理完成
await new Promise(resolve => setTimeout(resolve, 5000))

BatchProcessor.stopBatchProcessor()
```

### 工作池模式

```typescript
// 14. 工作池模式
class WorkerPool {
  private static workers: Map<string, worker.Worker> = new Map()
  private static taskQueue: Map<string, (() => Promise<void>)> = new Map()
  
  static async createWorker(workerName: string, scriptFile: string): Promise<worker.Worker> {
    if (this.workers.has(workerName)) {
      return this.workers.get(workerName)!
    }
    
    const wk = new worker.Worker(scriptFile)
    this.workers.set(workerName, wk)
    
    wk.onmessage = (event) => {
      const taskId = event.data.taskId
      const task = this.taskQueue.get(taskId)
      
      if (task) {
        task().catch(error => {
          console.error(`任务 ${taskId} 执行失败:`, error)
        }).finally(() => {
          this.taskQueue.delete(taskId)
        })
      }
    }
    
    wk.onerror = (error) => {
      console.error(`Worker ${workerName} 错误:`, error)
    }
    
    return wk
  }
  
  static async executeTask<T>(
    workerName: string,
    taskId: string,
    data: any
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      const task = async () => {
        try {
          const worker = this.workers.get(workerName)
          if (!worker) {
            throw new Error(`Worker ${workerName} 不存在`)
          }
          
          worker.postMessage({
            taskId: taskId,
            data: data
          })
        } catch (error) {
          reject(error)
        }
      }
      
      this.taskQueue.set(taskId, task as () => Promise<void>)
      task()
    })
  }
  
  static async terminateWorker(workerName: string): Promise<void> {
    const worker = this.workers.get(workerName)
    if (worker) {
      worker.terminate()
      this.workers.delete(workerName)
      console.log(`Worker ${workerName} 已终止`)
    }
  }
}

// 使用示例
// 创建Worker
const worker = await WorkerPool.createWorker(
  'imageProcessor',
  'workers/image-processor.js'
)

// 执行任务
const result1 = await WorkerPool.executeTask(
  'imageProcessor',
  'task1',
  { imageData: 'base64data' }
)

const result2 = await WorkerPool.executeTask(
  'imageProcessor',
  'task2',
  { imageData: 'base64data2' }
)

// 终止Worker
await WorkerPool.terminateWorker('imageProcessor')
```

## 错误处理与异常恢复

### 任务错误捕获

```typescript
// 15. 任务错误捕获
class ErrorHandlingTask {
  private static retryConfig: Map<string, RetryConfig> = new Map()
  
  static async executeWithRetry<T>(
    taskId: string,
    task: () => Promise<T>,
    maxRetries: number = 3,
    retryDelay: number = 1000
  ): Promise<T> {
    let lastError: Error | null = null
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        console.log(`任务 ${taskId} 第 ${attempt} 次尝试`)
        const result = await task()
        console.log(`任务 ${taskId} 第 ${attempt} 次尝试成功`)
        return result
      } catch (error) {
        lastError = error
        console.error(`任务 ${taskId} 第 ${attempt} 次尝试失败:`, error)
        
        if (attempt < maxRetries) {
          console.log(`等待 ${retryDelay}ms 后重试...`)
          await new Promise(resolve => setTimeout(resolve, retryDelay))
        }
      }
    }
    
    console.error(`任务 ${taskId} 所有重试均失败`)
    throw lastError!
  }
  
  static setRetryConfig(taskId: string, config: RetryConfig): void {
    this.retryConfig.set(taskId, config)
  }
}

interface RetryConfig {
  maxRetries: number
  retryDelay: number
  exponentialBackoff: boolean
  maxDelay: number
}

// 使用示例
const result = await ErrorHandlingTask.executeWithRetry(
  'dataFetch',
  async () => {
    console.log('获取数据...')
    const response = await fetch('https://api.example.com/data')
    return response.json()
  },
  3, // 最大重试3次
  1000 // 重试延迟1秒
)
```

### 任务超时与取消

```typescript
// 16. 任务超时与取消
class CancellableTask {
  private static cancelledTasks: Set<string> = new Set()
  
  static executeWithCancel<T>(
    taskId: string,
    task: () => Promise<T>,
    timeout: number
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      // 检查是否已取消
      if (this.cancelledTasks.has(taskId)) {
        reject(new Error(`任务 ${taskId} 已被取消`))
        return
      }
      
      // 设置超时
      const timeoutTimer = setTimeout(() => {
        if (!this.cancelledTasks.has(taskId)) {
          console.log(`任务 ${taskId} 超时 (${timeout}ms)`)
          ffrt.cancel(taskId)
          reject(new Error(`任务 ${taskId} 超时`))
        }
      }, timeout)
      
      // 提交任务
      const ffrtTask: ffrt.Task = {
        name: taskId,
        handler: async () => {
          try {
            const result = await task()
            
            // 清除超时计时器
            clearTimeout(timeoutTimer)
            
            // 检查是否被取消
            if (this.cancelledTasks.has(taskId)) {
              reject(new Error(`任务 ${taskId} 已被取消`))
              return
            }
            
            resolve(result)
          } catch (error) {
            clearTimeout(timeoutTimer)
            reject(error)
          }
        }
      }
      
      ffrt.submit(ffrtTask)
    })
  }
  
  static cancel(taskId: string): void {
    this.cancelledTasks.add(taskId)
    ffrt.cancel(taskId)
    console.log(`任务 ${taskId} 已取消`)
  }
}

// 使用示例
// 执行可取消的任务
const taskPromise = CancellableTask.executeWithCancel(
  'longRunningTask',
  async () => {
    console.log('开始长时间运行的任务')
    for (let i = 0; i < 10; i++) {
      await new Promise(resolve => setTimeout(resolve, 1000))
      console.log(`任务进度: ${(i + 1) * 10}%`)
    }
    console.log('长时间运行的任务完成')
    return 'Task Completed'
  },
  10000 // 10秒超时
)

// 模拟5秒后取消任务
setTimeout(() => {
  CancellableTask.cancel('longRunningTask')
}, 5000)
```

## 性能监控与调优

### 任务执行监控

```typescript
// 17. 任务执行监控
import hiTraceMeter from '@ohos.hiTraceMeter'

class TaskExecutionMonitor {
  private static metrics: Map<string, TaskMetrics> = new Map()
  
  static async executeWithMonitoring<T>(
    taskId: string,
    task: () => Promise<T>
  ): Promise<T> {
    const startTime = Date.now()
    hiTraceMeter.startTrace(`task-${taskId}`)
    
    try {
      const result = await task()
      
      const endTime = Date.now()
      const duration = endTime - startTime
      
      // 记录指标
      const metrics: TaskMetrics = {
        taskId: taskId,
        duration: duration,
        success: true,
        timestamp: new Date()
      }
      
      this.metrics.set(taskId, metrics)
      hiTraceMeter.finishTrace(`task-${taskId}`, false)
      
      // 性能分析
      if (duration > 1000) {
        console.warn(`任务 ${taskId} 耗时 ${duration}ms，超过1000ms阈值`)
      }
      
      return result
    } catch (error) {
      const endTime = Date.now()
      const duration = endTime - startTime
      
      const metrics: TaskMetrics = {
        taskId: taskId,
        duration: duration,
        success: false,
        error: error,
        timestamp: new Date()
      }
      
      this.metrics.set(taskId, metrics)
      hiTraceMeter.finishTrace(`task-${taskId}`, true)
      
      throw error
    }
  }
  
  static getMetrics(taskId: string): TaskMetrics | undefined {
    return this.metrics.get(taskId)
  }
  
  static getAllMetrics(): TaskMetrics[] {
    return Array.from(this.metrics.values())
  }
  
  static getAverageDuration(): number {
    const metrics = Array.from(this.metrics.values())
    if (metrics.length === 0) {
      return 0
    }
    
    const total = metrics.reduce((sum, m) => sum + m.duration, 0)
    return total / metrics.length
  }
}

interface TaskMetrics {
  taskId: string
  duration: number
  success: boolean
  error?: Error
  timestamp: Date
}

// 使用示例
const result = await TaskExecutionMonitor.executeWithMonitoring(
  'dataProcessing',
  async () => {
    console.log('处理数据')
    await new Promise(resolve => setTimeout(resolve, 800))
    return 'Processed Data'
  }
)

// 获取指标
const metrics = TaskExecutionMonitor.getMetrics('dataProcessing')
console.log('任务指标:', metrics)

// 获取平均执行时间
const avgDuration = TaskExecutionMonitor.getAverageDuration()
console.log('平均执行时间:', avgDuration, 'ms')
```

### 系统资源监控

```typescript
// 18. 系统资源监控
class SystemResourceMonitor {
  private static isMonitoring: boolean = false
  private static monitorInterval: NodeJS.Timeout | null = null
  private static cpuUsage: number[] = []
  private static memoryUsage: number[] = []
  
  static async startMonitoring(interval: number = 1000): Promise<void> {
    if (this.isMonitoring) {
      console.warn('系统资源监控已在运行')
      return
    }
    
    this.isMonitoring = true
    this.monitorInterval = setInterval(() => {
      this.collectMetrics()
    }, interval)
    
    console.log(`系统资源监控已启动，采样间隔: ${interval}ms`)
  }
  
  static stopMonitoring(): void {
    if (!this.isMonitoring) {
      return
    }
    
    if (this.monitorInterval) {
      clearInterval(this.monitorInterval)
      this.monitorInterval = null
    }
    
    this.isMonitoring = false
    console.log('系统资源监控已停止')
  }
  
  private static collectMetrics(): void {
    // 获取CPU使用率
    const cpuUsage = this.getCpuUsage()
    this.cpuUsage.push(cpuUsage)
    
    // 获取内存使用
    const memoryUsage = this.getMemoryUsage()
    this.memoryUsage.push(memoryUsage)
    
    console.log(`CPU使用率: ${cpuUsage.toFixed(1)}%, 内存使用: ${memoryUsage.toFixed(1)}MB`)
    
    // 性能告警
    if (cpuUsage > 80) {
      console.warn(`警告: CPU使用率过高: ${cpuUsage.toFixed(1)}%`)
    }
    
    if (memoryUsage > 200) {
      console.warn(`警告: 内存使用过高: ${memoryUsage.toFixed(1)}MB`)
    }
  }
  
  private static getCpuUsage(): number {
    // 模拟CPU使用率
    return Math.random() * 100
  }
  
  private static getMemoryUsage(): number {
    // 获取实际内存使用
    // 这里应该使用系统API获取
    return 100 + Math.random() * 100
  }
  
  static getAverageCpuUsage(): number {
    if (this.cpuUsage.length === 0) {
      return 0
    }
    
    const sum = this.cpuUsage.reduce((a, b) => a + b, 0)
    return sum / this.cpuUsage.length
  }
  
  static getAverageMemoryUsage(): number {
    if (this.memoryUsage.length === 0) {
      return 0
    }
    
    const sum = this.memoryUsage.reduce((a, b) => a + b, 0)
    return sum / this.memoryUsage.length
  }
}

// 使用示例
await SystemResourceMonitor.startMonitoring(2000)

// 执行一些任务...
await TaskExecutionMonitor.executeWithMonitoring('task1', someTask1)
await TaskExecutionMonitor.executeWithMonitoring('task2', someTask2)

// 停止监控
SystemResourceMonitor.stopMonitoring()

// 获取平均指标
const avgCpu = SystemResourceMonitor.getAverageCpuUsage()
const avgMemory = SystemResourceMonitor.getAverageMemoryUsage()

console.log(`平均CPU使用率: ${avgCpu.toFixed(1)}%`)
console.log(`平均内存使用: ${avgMemory.toFixed(1)}MB`)
```

## 最佳实践总结

### 1. 任务设计原则

- **任务粒度适中**：避免任务过大或过小
- **优先级合理设置**：关键任务使用高优先级
- **避免任务依赖环**：任务依赖关系应该是DAG（有向无环图）
- **及时释放资源**：任务完成后及时释放占用的资源

### 2. 并发控制策略

- **控制并发数量**：根据设备核心数合理设置并发数
- **使用任务池**：重用Worker，避免频繁创建销毁
- **实现负载均衡**：合理分配任务到不同核心
- **避免资源竞争**：注意线程安全和资源竞争

### 3. 错误处理策略

- **实现重试机制**：对可重试的错误进行重试
- **设置合理的超时**：避免任务无限期阻塞
- **记录详细日志**：便于问题排查和分析
- **优雅降级**：在任务失败时提供降级方案

### 4. 性能优化建议

- **使用HiTraceMeter**：监控关键任务的执行时间
- **定期分析指标**：收集和分析性能数据
- **优化热点任务**：优先优化耗时最长的任务
- **合理使用缓存**：减少重复计算和网络请求
- **异步化耗时操作**：避免阻塞主线程

## 参考

- [华为官方文档 - FFRT 概览](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ffrt-overview)
- [华为官方文档 - 并发范式](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ffrt-concurrency-paradigm)
- [华为官方文档 - 串行队列](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ffrt-concurrency-serial-queue)
- [华为官方文档 - 并发队列](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ffrt-concurrency-concurrent-queue)
- [华为官方文档 - 并行图](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ffrt-concurrency-graph)
- [华为官方文档 - API指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ffrt-api-guideline)
- [华为官方文档 - 开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ffrt-development-guideline)

> 注意：FFRT 框架的使用需要考虑设备的核心数和系统资源，合理设置并发参数，避免过度并发导致系统资源耗尽。
