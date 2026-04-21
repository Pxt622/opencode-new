# ArkTS 网络请求与数据存储

## HTTP 请求

```typescript
import http from '@ohos.net.http'

let httpRequest = http.createHttp()
httpRequest.request(
  'https://api.example.com/data',
  {
    method: http.RequestMethod.GET,
    header: {
      'Content-Type': 'application/json'
    }
  },
  (err, data) => {
    if (!err) {
      console.log('Result:', data.result)
    }
  }
)
```

## 异步请求

```typescript
async function fetchData() {
  try {
    let httpRequest = http.createHttp()
    const result = await httpRequest.request(
      'https://api.example.com/data',
      { method: http.RequestMethod.GET }
    )
    return JSON.parse(result.result as string)
  } catch (err) {
    console.error('Error:', err)
  }
}
```

## 首选项存储

```typescript
import dataPreferences from '@ohos.data.preferences'

let context = getContext(this)
let dataStore = await dataPreferences.getPreferences(context, 'myStore')

// 保存
await dataStore.put('name', 'Jack')
await dataStore.flush()

// 读取
const name = await dataStore.get('name', 'default')

// 删除
await dataStore.delete('name')
```

## 键值存储

```typescript
import kvStore from '@ohos.distributedKVStore'

let kvManager = kvStore.createKVManager(context)
let kvStore = await kvManager.getKVStore('store')

// 写入
await kvStore.put('key', 'value')

// 读取
const value = await kvStore.get('key')
```

## 关系型数据库

```typescript
import relationalStore from '@ohos.data.relationalStore'

// 数据库配置
const config: relationalStore.StoreConfig = {
  name: 'MyDatabase.db',
  securityLevel: relationalStore.SecurityLevel.S1
}

// 打开数据库
let store: relationalStore.RdbStore
try {
  store = await relationalStore.getRdbStore(context, config)
  
  // 创建表
  const sql = `CREATE TABLE IF NOT EXISTS user (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    age INTEGER,
    email TEXT UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
  )`
  
  await store.executeSql(sql)
} catch (error) {
  console.error('数据库操作失败:', error)
}

// 插入数据
async function insertUser(name: string, age: number, email: string): Promise<number> {
  const valueBucket: relationalStore.ValuesBucket = {
    name: name,
    age: age,
    email: email
  }
  
  return await store.insert('user', valueBucket)
}

// 查询数据
async function queryUsers(): Promise<any[]> {
  const predicates = new relationalStore.RdbPredicates('user')
  predicates.equalTo('age', 25) // 可选：添加查询条件
  
  const result = await store.query(predicates, ['id', 'name', 'age', 'email'])
  return result.getAllObjects()
}

// 更新数据
async function updateUser(id: number, updates: any): Promise<number> {
  const predicates = new relationalStore.RdbPredicates('user')
  predicates.equalTo('id', id)
  
  const valueBucket: relationalStore.ValuesBucket = updates
  return await store.update(valueBucket, predicates)
}

// 删除数据
async function deleteUser(id: number): Promise<number> {
  const predicates = new relationalStore.RdbPredicates('user')
  predicates.equalTo('id', id)
  
  return await store.delete(predicates)
}

// 事务处理
async function transferBalance(fromId: number, toId: number, amount: number): Promise<void> {
  await store.beginTransaction()
  
  try {
    // 扣除发送方余额
    const fromPredicates = new relationalStore.RdbPredicates('account')
    fromPredicates.equalTo('id', fromId)
    const fromAccount = await store.query(fromPredicates, ['balance'])
    const fromBalance = fromAccount.getColumnIndex('balance')
    
    if (fromBalance < amount) {
      throw new Error('余额不足')
    }
    
    await store.update(
      { balance: fromBalance - amount },
      fromPredicates
    )
    
    // 增加接收方余额
    const toPredicates = new relationalStore.RdbPredicates('account')
    toPredicates.equalTo('id', toId)
    const toAccount = await store.query(toPredicates, ['balance'])
    const toBalance = toAccount.getColumnIndex('balance')
    
    await store.update(
      { balance: toBalance + amount },
      toPredicates
    )
    
    await store.commit()
  } catch (error) {
    await store.rollback()
    throw error
  }
}
```

## 文件操作

```typescript
import fileio from '@ohos.fileio'
import picker from '@ohos.file.picker'

// 读取文件
async function readFile(filePath: string): Promise<string> {
  const fd = await fileio.open(filePath, fileio.OpenMode.READ_ONLY)
  const content = await fileio.readText(fd)
  await fileio.close(fd)
  return content
}

// 写入文件
async function writeFile(filePath: string, content: string): Promise<void> {
  const fd = await fileio.open(filePath, fileio.OpenMode.WRITE_ONLY | fileio.OpenMode.CREATE)
  await fileio.write(fd, content)
  await fileio.close(fd)
}

// 选择文件
async function pickFile(): Promise<string | null> {
  try {
    const filePicker = new picker.DocumentPicker()
    const result = await filePicker.select()
    
    if (result && result.length > 0) {
      return result[0].uri
    }
    return null
  } catch (error) {
    console.error('文件选择失败:', error)
    return null
  }
}

// 保存文件
async function saveFile(content: string, defaultName: string): Promise<string | null> {
  try {
    const fileSaver = new picker.DocumentSaver()
    const result = await fileSaver.save(defaultName, content)
    
    if (result) {
      return result.uri
    }
    return null
  } catch (error) {
    console.error('文件保存失败:', error)
    return null
  }
}

// 文件管理类
class FileManager {
  static async listFiles(dirPath: string): Promise<string[]> {
    try {
      const dir = await fileio.opendir(dirPath)
      const files: string[] = []
      
      let entry: fileio.Dirent
      while ((entry = await dir.read()) !== null) {
        files.push(entry.name)
      }
      
      await dir.close()
      return files
    } catch (error) {
      console.error('列出文件失败:', error)
      return []
    }
  }
  
  static async getFileInfo(filePath: string): Promise<fileio.Stat> {
    return await fileio.stat(filePath)
  }
  
  static async copyFile(source: string, destination: string): Promise<void> {
    await fileio.copyFile(source, destination)
  }
  
  static async moveFile(source: string, destination: string): Promise<void> {
    await fileio.moveFile(source, destination)
  }
  
  static async deleteFile(filePath: string): Promise<void> {
    await fileio.unlink(filePath)
  }
}
```

## WebSocket 通信

```typescript
import webSocket from '@ohos.net.webSocket'

class WebSocketClient {
  private socket: webSocket.WebSocket | null = null
  private url: string
  
  constructor(url: string) {
    this.url = url
  }
  
  // 连接WebSocket
  async connect(): Promise<void> {
    return new Promise((resolve, reject) => {
      this.socket = webSocket.createWebSocket()
      
      this.socket.on('open', () => {
        console.log('WebSocket连接已打开')
        resolve()
      })
      
      this.socket.on('message', (data: string | ArrayBuffer) => {
        console.log('收到消息:', data)
        this.onMessage(data)
      })
      
      this.socket.on('close', (code: number, reason: string) => {
        console.log(`WebSocket连接已关闭，代码: ${code}, 原因: ${reason}`)
        this.onClose(code, reason)
      })
      
      this.socket.on('error', (err: Error) => {
        console.error('WebSocket错误:', err)
        reject(err)
      })
      
      this.socket.connect(this.url)
    })
  }
  
  // 发送消息
  send(message: string | ArrayBuffer): void {
    if (this.socket && this.socket.readyState === webSocket.ReadyState.OPEN) {
      this.socket.send(message)
    } else {
      console.error('WebSocket未连接')
    }
  }
  
  // 关闭连接
  close(code?: number, reason?: string): void {
    if (this.socket) {
      this.socket.close(code, reason)
      this.socket = null
    }
  }
  
  // 消息处理回调
  onMessage(data: string | ArrayBuffer): void {
    // 子类可以重写此方法
    console.log('处理消息:', data)
  }
  
  // 连接关闭回调
  onClose(code: number, reason: string): void {
    // 子类可以重写此方法
    console.log(`连接关闭，代码: ${code}, 原因: ${reason}`)
  }
}

// 使用示例
const wsClient = new WebSocketClient('wss://echo.websocket.org')
await wsClient.connect()
wsClient.send('Hello WebSocket!')
```

## 网络状态检测

```typescript
import network from '@ohos.net.network'

class NetworkMonitor {
  private netHandle: network.NetHandle | null = null
  
  // 获取网络连接信息
  async getNetworkInfo(): Promise<network.ConnectionProperties> {
    this.netHandle = await network.getDefaultNet()
    
    if (!this.netHandle) {
      throw new Error('未获取到网络连接')
    }
    
    return await network.getConnectionProperties(this.netHandle)
  }
  
  // 检查网络是否可用
  async isNetworkAvailable(): Promise<boolean> {
    try {
      const info = await this.getNetworkInfo()
      return info.isAvailable === true
    } catch (error) {
      return false
    }
  }
  
  // 获取网络类型
  async getNetworkType(): Promise<string> {
    try {
      const info = await this.getNetworkInfo()
      return info.netCapabilities.bearerTypes.join(', ')
    } catch (error) {
      return 'unknown'
    }
  }
  
  // 监听网络变化
  registerNetworkCallback(): void {
    network.on('netAvailable', (data) => {
      console.log('网络可用:', data)
      this.onNetworkAvailable()
    })
    
    network.on('netLost', (data) => {
      console.log('网络丢失:', data)
      this.onNetworkLost()
    })
    
    network.on('netCapabilitiesChange', (data) => {
      console.log('网络能力变化:', data)
      this.onNetworkCapabilitiesChange()
    })
    
    network.on('netConnectionPropertiesChange', (data) => {
      console.log('网络连接属性变化:', data)
      this.onNetworkConnectionPropertiesChange()
    })
  }
  
  // 网络回调方法
  onNetworkAvailable(): void {
    console.log('网络已恢复，可以执行网络操作')
  }
  
  onNetworkLost(): void {
    console.log('网络丢失，暂停网络操作')
  }
  
  onNetworkCapabilitiesChange(): void {
    console.log('网络能力发生变化')
  }
  
  onNetworkConnectionPropertiesChange(): void {
    console.log('网络连接属性发生变化')
  }
}

// 使用示例
const monitor = new NetworkMonitor()
const isAvailable = await monitor.isNetworkAvailable()
console.log(`网络可用: ${isAvailable}`)

const networkType = await monitor.getNetworkType()
console.log(`网络类型: ${networkType}`)

monitor.registerNetworkCallback()
```

## 最佳实践

### 1. 网络请求最佳实践

- **错误处理**：所有网络请求都要有错误处理
- **超时设置**：为重要请求设置合理的超时时间
- **重试机制**：对可重试的失败实现重试逻辑
- **请求取消**：支持取消正在进行的请求
- **缓存策略**：合理使用缓存减少网络请求

### 2. 数据存储最佳实践

- **数据加密**：敏感数据在存储前进行加密
- **数据备份**：重要数据实现备份和恢复机制
- **存储限制**：注意设备的存储空间限制
- **数据迁移**：版本升级时考虑数据迁移
- **性能优化**：大数据量时使用分页和懒加载

### 3. 文件操作最佳实践

- **权限检查**：操作文件前检查权限
- **路径安全**：验证文件路径防止目录遍历攻击
- **资源释放**：及时关闭文件描述符
- **错误处理**：文件操作要有完善的错误处理
- **用户反馈**：大文件操作时提供进度反馈

### 4. WebSocket 最佳实践

- **心跳机制**：维持连接活跃
- **重连策略**：连接断开时自动重连
- **消息确认**：重要消息实现确认机制
- **流量控制**：避免发送过快导致阻塞
- **安全考虑**：使用 wss 协议和认证

## 文件上传下载

### HTTP 文件下载

```typescript
import http from '@ohos.net.http'
import fs from '@ohos.file.fs'
import { BusinessError } from '@ohos.base'

// 下载文件
async function downloadFile(url: string, savePath: string, onProgress?: (progress: number) => void): Promise<void> {
  try {
    const httpRequest = http.createHttp()
    
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.GET,
      expectDataType: http.HttpDataType.ARRAY_BUFFER,
      connectTimeout: 60000,
      readTimeout: 60000
    })
    
    if (response.responseCode === 200 && response.result instanceof ArrayBuffer) {
      // 写入文件
      const file = fs.openSync(savePath, fs.OpenMode.CREATE | fs.OpenMode.WRITE_ONLY)
      fs.writeSync(file.fd, response.result)
      fs.closeSync(file)
      
      console.log('文件下载成功:', savePath)
    } else {
      throw new Error(`下载失败，状态码: ${response.responseCode}`)
    }
    
    httpRequest.destroy()
  } catch (error) {
    const err = error as BusinessError
    console.error('下载文件失败:', err.message)
    throw err
  }
}

// 带进度的下载
async function downloadWithProgress(
  url: string,
  savePath: string,
  onProgress: (loaded: number, total: number, percentage: number) => void
): Promise<void> {
  try {
    const httpRequest = http.createHttp()
    
    httpRequest.on('dataReceive', (data: ArrayBuffer) => {
      const loaded = data.byteLength
      // 进度回调（简化版，实际需要跟踪总大小）
      onProgress(loaded, 0, 0)
    })
    
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.GET,
      expectDataType: http.HttpDataType.ARRAY_BUFFER
    })
    
    if (response.responseCode === 200 && response.result instanceof ArrayBuffer) {
      const file = fs.openSync(savePath, fs.OpenMode.CREATE | fs.OpenMode.WRITE_ONLY)
      fs.writeSync(file.fd, response.result)
      fs.closeSync(file)
      
      console.log('下载完成:', savePath)
    }
    
    httpRequest.destroy()
  } catch (error) {
    console.error('下载失败:', error)
    throw error
  }
}

// 使用示例
await downloadFile('https://example.com/file.zip', '/data/storage/el2/base/haps/entry/files/file.zip')

await downloadWithProgress(
  'https://example.com/large-file.zip',
  '/data/storage/el2/base/haps/entry/files/large-file.zip',
  (loaded, total, percentage) => {
    console.log(`下载进度: ${percentage}%`)
  }
)
```

### 断点续传下载

```typescript
class ResumableDownload {
  private downloadUrl: string
  private savePath: string
  private tempPath: string
  private totalSize: number = 0
  private downloadedSize: number = 0
  
  constructor(url: string, savePath: string) {
    this.downloadUrl = url
    this.savePath = savePath
    this.tempPath = `${savePath}.tmp`
  }
  
  // 获取文件大小
  private async getFileSize(): Promise<number> {
    const httpRequest = http.createHttp()
    const response = await httpRequest.request(this.downloadUrl, {
      method: http.RequestMethod.HEAD
    })
    httpRequest.destroy()
    
    const sizeHeader = response.header?.['Content-Length']
    return sizeHeader ? parseInt(sizeHeader) : 0
  }
  
  // 获取已下载大小
  private async getDownloadedSize(): Promise<number> {
    try {
      const stat = fs.statSync(this.tempPath)
      return stat.size
    } catch (error) {
      return 0
    }
  }
  
  // 开始/继续下载
  async download(onProgress?: (loaded: number, total: number, percentage: number) => void): Promise<void> {
    try {
      this.totalSize = await this.getFileSize()
      this.downloadedSize = await this.getDownloadedSize()
      
      console.log(`总大小: ${this.totalSize}, 已下载: ${this.downloadedSize}`)
      
      const httpRequest = http.createHttp()
      
      // 设置 Range 请求头实现断点续传
      const response = await httpRequest.request(this.downloadUrl, {
        method: http.RequestMethod.GET,
        header: {
          'Range': `bytes=${this.downloadedSize}-`
        },
        expectDataType: http.HttpDataType.ARRAY_BUFFER
      })
      
      if (response.responseCode === 206 && response.result instanceof ArrayBuffer) {
        // 追加写入文件
        const file = fs.openSync(
          this.tempPath,
          fs.OpenMode.CREATE | fs.OpenMode.WRITE_ONLY | fs.OpenMode.APPEND
        )
        fs.writeSync(file.fd, response.result)
        fs.closeSync(file)
        
        this.downloadedSize += response.result.byteLength
        
        // 触发进度回调
        if (onProgress) {
          onProgress(this.downloadedSize, this.totalSize, 
            Math.floor((this.downloadedSize / this.totalSize) * 100))
        }
        
        // 检查是否下载完成
        if (this.downloadedSize >= this.totalSize) {
          // 重命名为最终文件名
          fs.renameSync(this.tempPath, this.savePath)
          console.log('下载完成:', this.savePath)
        }
      }
      
      httpRequest.destroy()
    } catch (error) {
      console.error('下载失败:', error)
      throw error
    }
  }
  
  // 取消下载
  cancel(): void {
    console.log('下载已取消')
  }
}

// 使用示例
const downloader = new ResumableDownload(
  'https://example.com/large-file.zip',
  '/data/storage/el2/base/haps/entry/files/large-file.zip'
)

await downloader.download((loaded, total, percentage) => {
  console.log(`下载进度: ${percentage}% (${loaded}/${total})`)
})
```

### 文件上传

```typescript
import { FormData } from '@ohos.net.http'

// 上传单个文件
async function uploadFile(
  url: string,
  filePath: string,
  fieldName: string = 'file',
  onProgress?: (percentage: number) => void
): Promise<void> {
  try {
    const httpRequest = http.createHttp()
    
    // 读取文件内容
    const file = fs.openSync(filePath, fs.OpenMode.READ_ONLY)
    const stat = fs.statSync(filePath)
    const buffer = new ArrayBuffer(stat.size)
    fs.readSync(file.fd, buffer)
    fs.closeSync(file)
    
    // 创建 FormData
    const formData = new FormData()
    formData.append(fieldName, new Blob([buffer]), filePath.split('/').pop())
    
    // 监听上传进度
    httpRequest.on('dataSendProgress', (uploadedSize: number, totalSize: number) => {
      if (onProgress) {
        const percentage = Math.floor((uploadedSize / totalSize) * 100)
        onProgress(percentage)
      }
    })
    
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.POST,
      header: {
        'Content-Type': 'multipart/form-data'
      },
      extraData: formData
    })
    
    if (response.responseCode === 200) {
      console.log('上传成功:', response.result)
    } else {
      throw new Error(`上传失败，状态码: ${response.responseCode}`)
    }
    
    httpRequest.destroy()
  } catch (error) {
    console.error('上传文件失败:', error)
    throw error
  }
}

// 上传多个文件
async function uploadMultipleFiles(
  url: string,
  filePaths: string[],
  onProgress?: (percentage: number) => void
): Promise<void> {
  try {
    const httpRequest = http.createHttp()
    const formData = new FormData()
    
    // 添加所有文件
    for (let i = 0; i < filePaths.length; i++) {
      const filePath = filePaths[i]
      const file = fs.openSync(filePath, fs.OpenMode.READ_ONLY)
      const stat = fs.statSync(filePath)
      const buffer = new ArrayBuffer(stat.size)
      fs.readSync(file.fd, buffer)
      fs.closeSync(file)
      
      formData.append(`files[${i}]`, new Blob([buffer]), filePath.split('/').pop())
    }
    
    // 监听上传进度
    httpRequest.on('dataSendProgress', (uploadedSize: number, totalSize: number) => {
      if (onProgress) {
        const percentage = Math.floor((uploadedSize / totalSize) * 100)
        onProgress(percentage)
      }
    })
    
    const response = await httpRequest.request(url, {
      method: http.RequestMethod.POST,
      extraData: formData
    })
    
    if (response.responseCode === 200) {
      console.log('上传成功:', response.result)
    }
    
    httpRequest.destroy()
  } catch (error) {
    console.error('批量上传失败:', error)
    throw error
  }
}

// 使用示例
await uploadFile(
  'https://example.com/upload',
  '/data/storage/el2/base/haps/entry/files/photo.jpg',
  'photo',
  (percentage) => console.log(`上传进度: ${percentage}%`)
)

await uploadMultipleFiles(
  'https://example.com/batch-upload',
  [
    '/data/storage/el2/base/haps/entry/files/file1.jpg',
    '/data/storage/el2/base/haps/entry/files/file2.jpg',
    '/data/storage/el2/base/haps/entry/files/file3.jpg'
  ],
  (percentage) => console.log(`上传进度: ${percentage}%`)
)
```

## 数据压缩与解压缩

### zlib 压缩

```typescript
import zlib from '@ohos.zlib'

// 压缩字符串
async function compressString(text: string): Promise<ArrayBuffer> {
  try {
    const input = new TextEncoder().encodeInto(text, new Uint8Array(text.length))
    const compressed = await zlib.compress(input.buffer)
    
    console.log(`原始大小: ${text.length} bytes, 压缩后: ${compressed.byteLength} bytes`)
    return compressed
  } catch (error) {
    console.error('压缩失败:', error)
    throw error
  }
}

// 解压缩字符串
async function decompressString(compressedData: ArrayBuffer): Promise<string> {
  try {
    const decompressed = await zlib.decompress(compressedData)
    const text = new TextDecoder().decode(decompressed)
    
    return text
  } catch (error) {
    console.error('解压缩失败:', error)
    throw error
  }
}

// 压缩文件
async function compressFile(sourcePath: string, targetPath: string): Promise<void> {
  try {
    // 读取源文件
    const file = fs.openSync(sourcePath, fs.OpenMode.READ_ONLY)
    const stat = fs.statSync(sourcePath)
    const buffer = new ArrayBuffer(stat.size)
    fs.readSync(file.fd, buffer)
    fs.closeSync(file)
    
    // 压缩数据
    const compressed = await zlib.compress(buffer)
    
    // 写入目标文件
    const targetFile = fs.openSync(targetPath, fs.OpenMode.CREATE | fs.OpenMode.WRITE_ONLY)
    fs.writeSync(targetFile.fd, compressed)
    fs.closeSync(targetFile)
    
    console.log(`文件压缩完成: ${sourcePath} -> ${targetPath}`)
    console.log(`压缩率: ${Math.floor((1 - compressed.byteLength / stat.size) * 100)}%`)
  } catch (error) {
    console.error('压缩文件失败:', error)
    throw error
  }
}

// 解压缩文件
async function decompressFile(sourcePath: string, targetPath: string): Promise<void> {
  try {
    // 读取压缩文件
    const file = fs.openSync(sourcePath, fs.OpenMode.READ_ONLY)
    const stat = fs.statSync(sourcePath)
    const buffer = new ArrayBuffer(stat.size)
    fs.readSync(file.fd, buffer)
    fs.closeSync(file)
    
    // 解压缩数据
    const decompressed = await zlib.decompress(buffer)
    
    // 写入目标文件
    const targetFile = fs.openSync(targetPath, fs.OpenMode.CREATE | fs.OpenMode.WRITE_ONLY)
    fs.writeSync(targetFile.fd, decompressed)
    fs.closeSync(targetFile)
    
    console.log(`文件解压缩完成: ${sourcePath} -> ${targetPath}`)
  } catch (error) {
    console.error('解压缩文件失败:', error)
    throw error
  }
}

// 使用示例
const originalText = '这是一段很长的文本内容，需要进行压缩处理...'
const compressed = await compressString(originalText)
const decompressedText = await decompressString(compressed)
console.log('解压缩结果:', decompressedText)

await compressFile(
  '/data/storage/el2/base/haps/entry/files/large-data.json',
  '/data/storage/el2/base/haps/entry/files/large-data.json.zlib'
)

await decompressFile(
  '/data/storage/el2/base/haps/entry/files/large-data.json.zlib',
  '/data/storage/el2/base/haps/entry/files/large-data-restored.json'
)
```

### 压缩级别控制

```typescript
import zlib from '@ohos.zlib'

// 压缩选项
interface CompressOptions {
  level?: number // 压缩级别: 0-9，默认为 6
  memLevel?: number // 内存级别: 1-9，默认为 8
  strategy?: 0 | 1 | 2 // 压缩策略
}

// 带选项的压缩
async function compressWithOptions(
  data: string | ArrayBuffer,
  options: CompressOptions = {}
): Promise<ArrayBuffer> {
  try {
    let input: ArrayBuffer
    
    if (typeof data === 'string') {
      input = new TextEncoder().encode(data).buffer
    } else {
      input = data
    }
    
    const compressed = await zlib.compress(input, options)
    return compressed
  } catch (error) {
    console.error('压缩失败:', error)
    throw error
  }
}

// 快速压缩（速度优先）
async function fastCompress(data: string): Promise<ArrayBuffer> {
  return compressWithOptions(data, { level: 1 })
}

// 最佳压缩（大小优先）
async function bestCompress(data: string): Promise<ArrayBuffer> {
  return compressWithOptions(data, { level: 9 })
}

// 使用示例
const text = '这是一段很长的文本内容，需要进行压缩处理...'

// 快速压缩
const fastCompressed = await fastCompress(text)
console.log('快速压缩大小:', fastCompressed.byteLength)

// 默认压缩
const defaultCompressed = await compressWithOptions(text)
console.log('默认压缩大小:', defaultCompressed.byteLength)

// 最佳压缩
const bestCompressed = await bestCompress(text)
console.log('最佳压缩大小:', bestCompressed.byteLength)
```

## 剪贴板操作

### 系统剪贴板

```typescript
import { pasteboard } from '@kit.ArkKit'

// 复制文本到剪贴板
async function copyText(text: string): Promise<void> {
  try {
    const systemPasteboard = pasteboard.getSystemPasteboard()
    
    const pasteData = pasteboard.createData(pasteboard.MIMETYPE_TEXT_PLAIN, text)
    await systemPasteboard.setData(pasteData)
    
    console.log('文本已复制到剪贴板')
  } catch (error) {
    console.error('复制文本失败:', error)
    throw error
  }
}

// 从剪贴板粘贴文本
async function pasteText(): Promise<string> {
  try {
    const systemPasteboard = pasteboard.getSystemPasteboard()
    const pasteData = await systemPasteboard.getData()
    
    if (pasteData && pasteData.hasMimeType(pasteboard.MIMETYPE_TEXT_PLAIN)) {
      const text = pasteData.getPrimaryText()
      console.log('从剪贴板粘贴文本:', text)
      return text
    }
    
    return ''
  } catch (error) {
    console.error('粘贴文本失败:', error)
    throw error
  }
}

// 复制图片到剪贴板
async function copyImage(imageUri: string): Promise<void> {
  try {
    const systemPasteboard = pasteboard.getSystemPasteboard()
    
    const pasteData = pasteboard.createData(pasteboard.MIMETYPE_IMAGE_URI, imageUri)
    await systemPasteboard.setData(pasteData)
    
    console.log('图片已复制到剪贴板')
  } catch (error) {
    console.error('复制图片失败:', error)
    throw error
  }
}

// 从剪贴板获取图片
async function pasteImage(): Promise<string> {
  try {
    const systemPasteboard = pasteboard.getSystemPasteboard()
    const pasteData = await systemPasteboard.getData()
    
    if (pasteData && pasteData.hasMimeType(pasteboard.MIMETYPE_IMAGE_URI)) {
      const imageUri = pasteData.getPrimaryPixelMap() || pasteData.getPrimaryUri()
      console.log('从剪贴板粘贴图片:', imageUri)
      return imageUri || ''
    }
    
    return ''
  } catch (error) {
    console.error('粘贴图片失败:', error)
    throw error
  }
}

// 复制 HTML 内容到剪贴板
async function copyHtml(html: string): Promise<void> {
  try {
    const systemPasteboard = pasteboard.getSystemPasteboard()
    
    const pasteData = pasteboard.createData(pasteboard.MIMETYPE_TEXT_HTML, html)
    await systemPasteboard.setData(pasteData)
    
    console.log('HTML 已复制到剪贴板')
  } catch (error) {
    console.error('复制 HTML 失败:', error)
    throw error
  }
}

// 检查剪贴板是否有数据
async function hasPasteData(): Promise<boolean> {
  try {
    const systemPasteboard = pasteboard.getSystemPasteboard()
    return await systemPasteboard.hasPasteData()
  } catch (error) {
    console.error('检查剪贴板失败:', error)
    return false
  }
}

// 清空剪贴板
async function clearClipboard(): Promise<void> {
  try {
    const systemPasteboard = pasteboard.getSystemPasteboard()
    await systemPasteboard.clear()
    console.log('剪贴板已清空')
  } catch (error) {
    console.error('清空剪贴板失败:', error)
    throw error
  }
}

// 使用示例
await copyText('Hello, World!')
const pastedText = await pasteText()
console.log('粘贴的文本:', pastedText)

await copyImage('/data/storage/el2/base/haps/entry/files/photo.png')
const pastedImage = await pasteImage()
console.log('粘贴的图片:', pastedImage)

const hasData = await hasPasteData()
console.log('剪贴板是否有数据:', hasData)

await clearClipboard()
```

### 剪贴板监听

```typescript
import { pasteboard } from '@kit.ArkKit'

// 剪贴板监听器
class ClipboardMonitor {
  private listenerId: number = 0
  private callback: (data: string) => void
  
  constructor(callback: (data: string) => void) {
    this.callback = callback
  }
  
  // 开始监听
  startMonitoring(): void {
    const systemPasteboard = pasteboard.getSystemPasteboard()
    
    this.listenerId = systemPasteboard.on('contentChange', async () => {
      try {
        const pasteData = await systemPasteboard.getData()
        if (pasteData && pasteData.hasMimeType(pasteboard.MIMETYPE_TEXT_PLAIN)) {
          const text = pasteData.getPrimaryText()
          this.callback(text)
        }
      } catch (error) {
        console.error('监听剪贴板变化失败:', error)
      }
    })
    
    console.log('剪贴板监听已启动')
  }
  
  // 停止监听
  stopMonitoring(): void {
    if (this.listenerId !== 0) {
      const systemPasteboard = pasteboard.getSystemPasteboard()
      systemPasteboard.off('contentChange', this.listenerId)
      this.listenerId = 0
      console.log('剪贴板监听已停止')
    }
  }
}

// 使用示例
const monitor = new ClipboardMonitor((text) => {
  console.log('剪贴板内容变化:', text)
  // 可以在这里实现自动处理逻辑
})

monitor.startMonitoring()

// 适当时停止监听
// onDestroy() {
//   monitor.stopMonitoring()
// }
```

## URL 路径处理

### URL 解析与构建

```typescript
import { URL } from '@kit.ArkKit'

// 解析 URL
function parseUrl(urlString: string): URL {
  const url = new URL(urlString)
  
  console.log('协议:', url.protocol)
  console.log('主机:', url.hostname)
  console.log('端口:', url.port)
  console.log('路径:', url.pathname)
  console.log('查询参数:', url.search)
  console.log('锚点:', url.hash)
  
  return url
}

// 构建 URL
function buildUrl(options: {
  protocol: string
  hostname: string
  port?: string
  pathname?: string
  searchParams?: Record<string, string>
  hash?: string
}): string {
  const url = new URL(`${options.protocol}//${options.hostname}`)
  
  if (options.port) {
    url.port = options.port
  }
  
  if (options.pathname) {
    url.pathname = options.pathname
  }
  
  if (options.searchParams) {
    Object.entries(options.searchParams).forEach(([key, value]) => {
      url.searchParams.append(key, value)
    })
  }
  
  if (options.hash) {
    url.hash = options.hash
  }
  
  return url.toString()
}

// 使用示例
const url = parseUrl('https://api.example.com:8080/users?page=1&limit=20#section1')

const newUrl = buildUrl({
  protocol: 'https:',
  hostname: 'api.example.com',
  port: '8080',
  pathname: '/users',
  searchParams: {
    page: '1',
    limit: '20'
  },
  hash: 'section1'
})
console.log('构建的 URL:', newUrl)
```

### 查询参数处理

```typescript
import { URLSearchParams } from '@kit.ArkKit'

// 解析查询参数
function parseQueryString(queryString: string): Record<string, string> {
  const params = new URLSearchParams(queryString)
  const result: Record<string, string> = {}
  
  params.forEach((value, key) => {
    result[key] = value
  })
  
  return result
}

// 构建查询参数
function buildQueryString(params: Record<string, string | number>): string {
  const searchParams = new URLSearchParams()
  
  Object.entries(params).forEach(([key, value]) => {
    searchParams.append(key, String(value))
  })
  
  return searchParams.toString()
}

// 更新查询参数
function updateQueryString(
  originalUrl: string,
  updates: Record<string, string>
): string {
  const url = new URL(originalUrl)
  
  Object.entries(updates).forEach(([key, value]) => {
    url.searchParams.set(key, value)
  })
  
  return url.toString()
}

// 使用示例
const queryString = 'page=1&limit=20&sort=desc'
const params = parseQueryString(queryString)
console.log('解析的参数:', params)

const newQueryString = buildQueryString({
  page: 2,
  limit: 30,
  sort: 'asc',
  filter: 'active'
})
console.log('构建的查询字符串:', newQueryString)

const updatedUrl = updateQueryString(
  'https://api.example.com/users?page=1&limit=20',
  { page: '2', filter: 'active' }
)
console.log('更新的 URL:', updatedUrl)
```

### URL 编码与解码

```typescript
// URL 编码
function encodeUrlComponent(str: string): string {
  return encodeURIComponent(str)
}

// URL 解码
function decodeUrlComponent(encodedStr: string): string {
  return decodeURIComponent(encodedStr)
}

// 编码整个 URL
function encodeFullUrl(url: string): string {
  return encodeURI(url)
}

// 解码整个 URL
function decodeFullUrl(encodedUrl: string): string {
  return decodeURI(encodedUrl)
}

// 构建带编码参数的 URL
function buildUrlWithParams(baseUrl: string, params: Record<string, any>): string {
  const searchParams = new URLSearchParams()
  
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null) {
      searchParams.append(key, encodeURIComponent(String(value)))
    }
  })
  
  const queryString = searchParams.toString()
  return queryString ? `${baseUrl}?${queryString}` : baseUrl
}

// 使用示例
const unsafeString = 'Hello World! 你好世界！'
const encoded = encodeUrlComponent(unsafeString)
console.log('编码结果:', encoded) // Hello%20World%21%20%E4%BD%A0%E5%A5%BD%E4%B8%96%E7%95%8C%EF%BC%81

const decoded = decodeUrlComponent(encoded)
console.log('解码结果:', decoded) // Hello World! 你好世界！

const url = buildUrlWithParams('https://api.example.com/search', {
  q: 'HarmonyOS 开发',
  page: 1,
  limit: 20,
  lang: 'zh-CN'
})
console.log('构建的 URL:', url)
```

### 路径处理

```typescript
// 规范化路径
function normalizePath(path: string): string {
  // 移除多余的斜杠
  return path.replace(/\/+/g, '/')
}

// 连接路径
function joinPaths(...paths: string[]): string {
  return paths
    .map(p => p.replace(/^\/+|\/+$/g, '')) // 移除首尾斜杠
    .filter(p => p.length > 0)
    .join('/')
}

// 获取路径的目录部分
function getDirname(path: string): string {
  const parts = path.split('/')
  parts.pop()
  return parts.join('/') || '/'
}

// 获取路径的文件名部分
function getBasename(path: string): string {
  const parts = path.split('/')
  return parts[parts.length - 1] || ''
}

// 获取文件扩展名
function getExtension(filename: string): string {
  const parts = filename.split('.')
  return parts.length > 1 ? parts[parts.length - 1] : ''
}

// 判断是否是绝对路径
function isAbsolutePath(path: string): boolean {
  return path.startsWith('/')
}

// 使用示例
console.log('规范化路径:', normalizePath('a//b///c')) // a/b/c
console.log('连接路径:', joinPaths('api', 'v1', 'users')) // api/v1/users
console.log('目录:', getDirname('/api/v1/users')) // /api/v1
console.log('文件名:', getBasename('/api/v1/users.json')) // users.json
console.log('扩展名:', getExtension('data.json.gz')) // gz
console.log('是否绝对路径:', isAbsolutePath('/data/file.txt')) // true
```

## 参考

- [华为官方文档 - 网络连接](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-net-http)
- [华为官方文档 - 数据存储](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-data-preferences)
- [华为官方文档 - 关系型数据库](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-data-relationalstore)
- [华为官方文档 - 文件管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-fileio)
- [华为官方文档 - WebSocket](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-net-webSocket)
- [华为官方文档 - 压缩解压缩](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-zlib)
- [华为官方文档 - 剪贴板](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-pasteboard)
- [华为官方文档 - URL](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-url)
