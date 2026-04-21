# HarmonyOS 分布式交互与跨端协同指南

## 分布式交互概览

HarmonyOS 的分布式能力实现了设备间的无缝协同，包括数据共享、任务迁移、界面流转等。本指南详细讲解跨端文件分享、拖拽操作、剪贴板等分布式交互功能的实现方法。

## 核心能力矩阵

| 能力 | API | 应用场景 | 权限要求 |
|------|-----|---------|---------|
| **文件分享** | `@ohos.share.distributedData` | 跨设备文件传输 | `ohos.permission.DISTRIBUTED_DATASYNC` |
| **拖拽上传** | `@ohos.ability.drag` | 文件拖拽到其他应用 | `ohos.permission.READ_WRITE_DOWNLOAD` |
| **跨端拖拽** | `跨设备拖拽API` | 跨设备文件拖拽 | `ohos.permission.DISTRIBUTED_DEVICE_STATE` |
| **剪贴板** | `@ohos.system.distributedPasteboard` | 跨设备剪贴板 | `ohos.permission.DISTRIBUTED_DATASYNC` |
| **跨端启动** | `startAbility` with deviceId | 跨设备启动Ability | `ohos.permission.INTERACT_ACROSS_DEVICES` |

## 应用文件分享

### 基础实现

```typescript
import distributedData from '@ohos.data.distributedData'

class FileShareManager {
  private static kvManager: distributedData.KVManager | null = null
  private static kvStore: distributedData.SingleKVStore | null = null
  
  static async initialize(): Promise<void> {
    try {
      // 创建分布式数据管理器
      this.kvManager = distributedData.createKVManager(getContext(this))
      
      // 创建单版本分布式数据库
      const options: distributedData.Options = {
        createIfMissing: true,
        securityLevel: distributedData.SecurityLevel.S1
      }
      
      this.kvStore = await this.kvManager.getKVStore('FileShare', options)
      
      console.log('文件分享管理器初始化成功')
    } catch (error) {
      console.error('初始化失败:', error)
      throw error
    }
  }
  
  static async shareFile(
    fileInfo: FileInfo,
    targetDeviceId?: string
  ): Promise<void> {
    try {
      // 1. 生成分享ID
      const shareId = this.generateShareId(fileInfo)
      
      // 2. 存储文件信息到分布式数据库
      const shareData: ShareData = {
        shareId: shareId,
        fileName: fileInfo.name,
        fileSize: fileInfo.size,
        fileType: fileInfo.type,
        filePath: fileInfo.path,
        sharedBy: 'MyApp',
        sharedAt: Date.now(),
        expiresAt: Date.now() + 24 * 60 * 60 * 1000 // 24小时过期
      }
      
      await this.kvStore.put(`share_${shareId}`, JSON.stringify(shareData))
      
      // 3. 生成分享内容
      const shareContent = await this.generateShareContent(shareData, targetDeviceId)
      
      // 4. 调用系统分享功能
      const shareController = new share.ShareController({
        data: shareContent,
        contentType: 'text/plain',
        title: `分享文件: ${fileInfo.name}`,
        mode: share.ShareMode.DistributedData
      })
      
      // 5. 显示分享面板
      shareController.show(this.getUIContext())
      
      console.log('文件分享成功:', fileInfo.name)
    } catch (error) {
      console.error('文件分享失败:', error)
      throw error
    }
  }
  
  static async getSharedFiles(): Promise<ShareData[]> {
    try {
      const allKeys = await this.kvStore.get('all_keys', '') as string[]
      const sharedFiles: ShareData[] = []
      
      for (const key of allKeys) {
        if (key.startsWith('share_')) {
          const value = await this.kvStore.get(key, '') as string
          const shareData = JSON.parse(value) as ShareData
          
          // 过滤过期和当前设备分享的文件
          if (shareData.expiresAt > Date.now() && shareData.sharedBy !== 'MyApp') {
            sharedFiles.push(shareData)
          }
        }
      }
      
      // 按分享时间倒序排列
      sharedFiles.sort((a, b) => b.sharedAt - a.sharedAt)
      
      return sharedFiles
    } catch (error) {
      console.error('获取分享文件失败:', error)
      return []
    }
  }
  
  private static generateShareId(fileInfo: FileInfo): string {
    const timestamp = Date.now()
    const hash = this.simpleHash(fileInfo.name + timestamp)
    return `share_${hash}`
  }
  
  private static simpleHash(str: string): string {
    let hash = 0
    for (let i = 0; i < str.length; i++) {
      hash = ((hash << 5) - hash) + str.charCodeAt(i)
      hash = hash & hash
    }
    return Math.abs(hash).toString(36)
  }
  
  private static async generateShareContent(
    shareData: ShareData,
    targetDeviceId?: string
  ): Promise<string> {
    // 这里应该生成实际的分享内容
    // 例如：文件预览链接、文件描述等
    return `文件分享: ${shareData.fileName} (${this.formatFileSize(shareData.fileSize)})`
  }
  
  private static formatFileSize(bytes: number): string {
    if (bytes < 1024) return `${bytes}B`
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(2)}KB`
    return `${(bytes / (1024 * 1024)).toFixed(2)}MB`
  }
}

// 数据接口定义
interface FileInfo {
  name: string
  path: string
  size: number
  type: string
}

interface ShareData {
  shareId: string
  fileName: string
  fileSize: number
  fileType: string
  filePath: string
  sharedBy: string
  sharedAt: number
  expiresAt: number
}

// 使用示例
@Entry
@Component
struct FileSharePage {
  @State sharedFiles: ShareData[] = []
  @State selectedFiles: string[] = []
  
  async aboutToAppear() {
    await FileShareManager.initialize()
    await this.loadSharedFiles()
  }
  
  async loadSharedFiles() {
    this.sharedFiles = await FileShareManager.getSharedFiles()
  }
  
  async selectFile(shareId: string) {
    if (this.selectedFiles.includes(shareId)) {
      this.selectedFiles = this.selectedFiles.filter(id => id !== shareId)
    } else {
      this.selectedFiles.push(shareId)
    }
  }
  
  build() {
    Column() {
      Text('文件分享')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      
      List() {
        ForEach(this.sharedFiles, (file: ShareData) => {
          ListItem() {
            FileCard({
              file: file,
              isSelected: this.selectedFiles.includes(file.shareId),
              onSelect: (shareId: string) => this.selectFile(shareId)
            })
          }
        }, (file: ShareData) => file.shareId)
      }
      
      Button('分享新文件')
        .onClick(() => {
          // 这里实现文件选择和分享逻辑
        })
    }
  }
}

@Component
struct FileCard {
  @Prop file: ShareData
  @Prop isSelected: boolean
  onSelect: (shareId: string) => void = () => {}
  
  build() {
    Row() {
      Column() {
        Text(this.file.fileName)
          .fontSize(16)
          .fontWeight(FontWeight.Medium)
        
        Text(this.formatFileSize(this.file.fileSize))
          .fontSize(14)
          .fontColor('#999999')
        
        Text(`来自: ${this.file.sharedBy}`)
          .fontSize(12)
          .fontColor('#666666')
      }
      .width('70%')
    }
    .padding(16)
    .backgroundColor(this.isSelected ? '#F0F0F0' : '#FFFFFF')
    .borderRadius(8)
    .onClick(() => {
      this.onSelect(this.file.shareId)
    })
  }
  
  private formatFileSize(bytes: number): string {
    if (bytes < 1024) return `${bytes}B`
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(2)}KB`
    return `${(bytes / (1024 * 1024)).toFixed(2)}MB`
  }
}

import share from '@ohos.share'
```

## 跨端拖拽上传

### 拖拽实现

```typescript
import { drag } from '@ohos.ability.drag'

class DragUploadManager {
  private static dragController: drag.DragController | null = null
  private static draggedFiles: FileInfo[] = []
  
  static initialize(): void {
    this.dragController = new drag.DragController()
    
    // 设置拖拽事件监听
    this.setupDragListeners()
  }
  
  private static setupDragListeners(): void {
    // 拖拽开始
    this.dragController.on('dragStart', (event: DragEvent) => {
      console.log('拖拽开始:', event)
      this.handleDragStart(event)
    })
    
    // 拖拽过程
    this.dragController.on('drag', (event: DragEvent) => {
      this.handleDragMove(event)
    })
    
    // 拖拽结束
    this.dragController.on('drop', (event: DragEvent) => {
      console.log('拖拽结束:', event)
      this.handleDrop(event)
    })
  }
  
  private static handleDragStart(event: DragEvent): void {
    // 这里可以初始化拖拽状态
    console.log('准备拖拽:', event)
  }
  
  private static handleDragMove(event: DragEvent): void {
    // 更新拖拽位置或预览
    console.log('拖拽中:', event)
  }
  
  private static async handleDrop(event: DropEvent): Promise<void> {
    try {
      // 1. 获取拖拽的文件信息
      const dropFiles = await this.getDropFiles(event)
      
      // 2. 处理文件
      for (const file of dropFiles) {
        await this.processDroppedFile(file)
      }
      
      console.log('文件拖拽处理完成:', dropFiles.length, '个文件')
    } catch (error) {
      console.error('处理拖拽文件失败:', error)
    }
  }
  
  private static async getDropFiles(event: DropEvent): Promise<FileInfo[]> {
    // 这里应该从事件中获取实际的文件信息
    // 实际实现中需要根据具体的拖拽事件来解析
    return []
  }
  
  private static async processDroppedFile(file: FileInfo): Promise<void> {
    // 这里实现文件处理逻辑
    console.log('处理文件:', file.name)
    
    // 可以：
    // 1. 上传到服务器
    // 2. 保存到本地
    // 3. 转换格式等
  }
}

// 事件类型定义
interface DragEvent {
  x: number
  y: number
  type: string
  data: any
}

interface DropEvent extends DragEvent {
  // 添加拖拽结束特有的属性
}

// 使用示例
@Entry
@Component
struct DragUploadPage {
  @State files: FileInfo[] = []
  @State isDragging: boolean = false
  
  aboutToAppear() {
    DragUploadManager.initialize()
    this.setupDragStateListeners()
  }
  
  build() {
    Column() {
      Text('拖拽文件到此处上传')
        .fontSize(18)
        .fontColor('#999999')
      
      // 拖拽区域
      Column() {
        if (this.isDragging) {
          Text('释放文件以上传')
            .fontSize(24)
            .fontColor('#CCCCCC')
        } else {
          Text('拖拽文件到此处')
            .fontSize(18)
            .fontColor('#666666')
        }
        
        // 拖拽预览
        if (this.files.length > 0) {
          Divider()
          
          ForEach(this.files, (file: FileInfo) => {
            FilePreviewItem({ file: file })
          }, (file: FileInfo) => file.name)
        }
      }
      .width('100%')
      .height(300)
      .backgroundColor('#F5F5F5')
      .borderRadius(12)
      .borderStyle({
        width: 2,
        color: this.isDragging ? '#4CAF50' : '#DDDDDD'
      })
    }
    .padding(20)
    .margin({ top: 20 })
  }
  
  private setupDragStateListeners(): void {
    // 这里设置拖拽状态变化的监听
    // 实际实现中需要与拖拽管理器集成
  }
}

@Component
struct FilePreviewItem {
  @Prop file: FileInfo
  
  build() {
    Row() {
      Text('📄')
        .fontSize(20)
      .margin({ right: 8 })
      
      Column() {
        Text(this.file.name)
          .fontSize(14)
          .fontWeight(FontWeight.Medium)
        
        Text(this.formatFileSize(this.file.size))
          .fontSize(12)
          .fontColor('#999999')
      }
      .alignItems(VerticalAlign.Center)
    }
    .padding(12)
    .backgroundColor('#FFFFFF')
    .borderRadius(8)
    .shadow({ radius: 4, color: '#00000020' })
  }
  
  private formatFileSize(bytes: number): string {
    if (bytes < 1024) return `${bytes}B`
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(2)}KB`
    return `${(bytes / (1024 * 1024)).toFixed(2)}MB`
  }
}
```

## 跨设备拖拽

### 分布式拖拽实现

```typescript
import deviceManager from '@ohos.distributedHardware.deviceManager'

class DistributedDragManager {
  private static deviceManager: deviceManager.DeviceManager | null = null
  private static targetDevices: deviceManager.DeviceInfo[] = []
  
  static async initialize(): Promise<void> {
    try {
      this.deviceManager = deviceManager.createDeviceManager(getContext(this))
      
      // 获取在线设备列表
      this.targetDevices = await this.getOnlineDevices()
      
      console.log('分布式拖拽管理器初始化完成，可用设备数:', this.targetDevices.length)
    } catch (error) {
      console.error('初始化分布式拖拽管理器失败:', error)
      throw error
    }
  }
  
  private static async getOnlineDevices(): Promise<deviceManager.DeviceInfo[]> {
    const allDevices = await this.deviceManager.getTrustedDeviceList()
    return allDevices.filter(device => device.deviceState === deviceManager.DeviceState.STATE_ONLINE)
  }
  
  static async dragToDevice(
    fileInfo: FileInfo,
    targetDeviceId: string
  ): Promise<void> {
    try {
      // 1. 准备拖拽数据
      const dragData: DistributedDragData = {
        fileName: fileInfo.name,
        fileSize: fileInfo.size,
        filePath: fileInfo.path,
        fileType: fileInfo.type,
        deviceId: targetDeviceId
      }
      
      // 2. 创建拖拽事件
      const dragEvent: DistributedDragEvent = {
        type: 'distributed.drag',
        data: dragData
      }
      
      // 3. 触发跨设备拖拽
      const dragController = new drag.DragController()
      dragController.on('drop', (event: DistributedDropEvent) => {
        this.handleDistributedDrop(event)
      })
      
      // 4. 显示设备选择界面
      this.showDeviceSelector(dragData, targetDeviceId)
      
      console.log('跨设备拖拽已启动:', fileInfo.name, '→', targetDeviceId)
    } catch (error) {
      console.error('跨设备拖拽失败:', error)
      throw error
    }
  }
  
  private static showDeviceSelector(
    dragData: DistributedDragData,
    preSelectedDeviceId?: string
  ): void {
    // 这里应该显示设备选择界面
    // 用户选择目标设备后执行拖拽
    console.log('显示设备选择器')
    console.log('预选设备:', preSelectedDeviceId)
  }
  
  private static handleDistributedDrop(event: DistributedDropEvent): void {
    console.log('跨设备拖拽完成:', event)
    // 处理拖拽完成后的逻辑
  }
}

interface DistributedDragData {
  fileName: string
  fileSize: number
  filePath: string
  fileType: string
  deviceId: string
}

interface DistributedDragEvent extends DragEvent {
  // 添加跨设备拖拽特有的属性
}
```

## 统一拖拽框架

### 统一拖拽实现

```typescript
import { unifiedData } from '@ohos.data.unifiedData'

class UnifiedDragAndDrop {
  private static unifiedData: unifiedData.UnifiedData | null = null
  private static dragController: drag.DragController | null = null
  
  static async initialize(): Promise<void> {
    this.unifiedData = unifiedData.createUnifiedData(getContext(this))
    this.dragController = new drag.DragController()
    
    console.log('统一拖拽框架初始化完成')
  }
  
  static async setupUnifiedDrag(
    uri: string,
    unifiedType: unifiedData.UnifiedType
  ): Promise<void> {
    try {
      // 1. 创建统一数据描述
      const data = await this.unifiedData.createData({
        // 数据类型：纯文本、文件、图片等
        unifiedType: unifiedType,
        // URI或数据内容
        uri: uri,
        // 额览信息
        description: `统一拖拽数据: ${unifiedType}`
      })
      
      // 2. 创建拖拽信息
      const dragInfo: UnifiedDragInfo = {
        data: data,
        preview: this.generatePreview(data),
        extras: this.createExtras()
      }
      
      // 3. 设置拖拽事件
      this.setupUnifiedDragEvents()
      
      console.log('统一拖拽已设置:', unifiedType)
    } catch (error) {
      console.error('设置统一拖拽失败:', error)
      throw error
    }
  }
  
  private static generatePreview(data: any): Preview {
    // 根据数据类型生成预览
    if (data.unifiedType === unifiedData.UnifiedType.TEXT) {
      return {
        type: 'text',
        content: data.record as string
      }
    } else if (data.unifiedType === unifiedData.UnifiedType.FILE) {
      return {
        type: 'file',
        fileName: data.details?.fileName || 'unknown'
      }
    } else if (data.unifiedType === unifiedData.UnifiedType.IMAGE) {
      return {
        type: 'image',
        pixelMap: data.record as image.PixelMap
      }
    }
    
    return { type: 'text', content: '' }
  }
  
  private static createExtras(): Record<string, string> {
    return {
      appId: 'com.example.myapp',
      appName: 'MyApp',
      version: '1.0.0'
    }
  }
  
  private static setupUnifiedDragEvents(): void {
    if (!this.dragController) return
    
    // 统一拖拽事件监听
    this.dragController.on('dragStart', (event) => {
      console.log('统一拖拽开始:', event)
    })
    
    this.dragController.on('drop', (event) => {
      console.log('统一拖拽结束:', event)
    })
  }
}

interface Preview {
  type: 'text' | 'file' | 'image'
  content?: string
  fileName?: string
  pixelMap?: image.PixelMap
}

interface UnifiedDragInfo {
  data: any
  preview: Preview
  extras: Record<string, string>
}
```

## 剪贴板同步

### 跨设备剪贴板

```typescript
import { systemPasteboard } from '@ohos.system.distributedPasteboard'

class DistributedClipboardManager {
  private static pasteboard: systemPasteboard.SystemPasteboard | null = null
  
  static async initialize(): Promise<void> {
    try {
      this.pasteboard = systemPasteboard.createSystemPasteboard(getContext(this))
      
      // 监听剪贴板内容变化
      this.setupClipboardListeners()
      
      console.log('分布式剪贴板管理器初始化完成')
    } catch (error) {
      console.error('初始化分布式剪贴板失败:', error)
      throw error
    }
  }
  
  private static setupClipboardListeners(): void {
    if (!this.pasteboard) return
    
    // 剪贴板内容变化监听
    this.pasteboard.on('contentChange', (content: string) => {
      console.log('剪贴板内容变化:', content)
      this.handleContentChange(content)
    })
  }
  
  private static handleContentChange(content: string): void {
    // 处理剪贴板内容变化
    console.log('处理新剪贴板内容')
    
    // 可以根据内容类型做不同处理
    if (this.isImageUrl(content)) {
      this.handleImagePaste(content)
    } else if (this.isFileContent(content)) {
      this.handleFilePaste(content)
    } else {
      this.handleTextPaste(content)
    }
  }
  
  static async setText(text: string): Promise<void> {
    try {
      await this.pasteboard.setData(systemPasteboard.SystemPasteboardData.TEXT, text)
      console.log('剪贴板文本设置成功')
    } catch (error) {
      console.error('设置剪贴板文本失败:', error)
      throw error
    }
  }
  
  static async setImage(image: image.PixelMap): Promise<void> {
    try {
      await this.pasteboard.setData(systemPasteboard.SystemPasteboardData.IMAGE, image)
      console.log('剪贴板图片设置成功')
    } catch (error) {
      console.error('设置剪贴板图片失败:', error)
      throw error
    }
  }
  
  static async getData<T>(
    dataKind: systemPasteboard.SystemPasteboardData
  ): Promise<T> {
    try {
      const result = await this.pasteboard.getData(dataKind)
      return result as T
    } catch (error) {
      console.error('获取剪贴板数据失败:', error)
      throw error
    }
  }
  
  private static isImageUrl(content: string): boolean {
    return content.startsWith('data:image/')
  }
  
  private static isFileContent(content: string): boolean {
    // 这里实现文件内容检测逻辑
    return false
  }
  
  private static handleImagePaste(content: string): void {
    console.log('处理图片粘贴:', content.substring(0, 50) + '...')
  }
  
  private static handleFilePaste(content: string): void {
    console.log('处理文件粘贴:', content.substring(0, 50) + '...')
  }
  
  private static handleTextPaste(content: string): void {
    console.log('处理文本粘贴:', content.substring(0, 50) + '...')
  }
}

// 使用示例
@Entry
@Component
struct ClipboardPage {
  @State clipboardContent: string = ''
  @State lastUpdate: string = ''
  
  async aboutToAppear() {
    await DistributedClipboardManager.initialize()
    await this.updateClipboardContent()
  }
  
  async updateClipboardContent(): Promise<void> {
    try {
      // 获取文本内容
      const text = await DistributedClipboardManager.getData<string>(
        systemPasteboard.SystemPasteboardData.TEXT
      )
      this.clipboardContent = text
      this.lastUpdate = this.formatTime(new Date())
    } catch (error) {
      console.error('获取剪贴板内容失败:', error)
    }
  }
  
  build() {
    Column() {
      Text('跨设备剪贴板')
        .fontSize(20)
        .fontWeight(FontBold)
      
      Divider()
      
      Row() {
        Text('剪贴板内容:')
          .fontSize(16)
          .fontWeight(FontWeight.Medium)
          .margin({ right: 16 })
        
        Text(`更新于: ${this.lastUpdate}`)
          .fontSize(12)
          .fontColor('#999999')
      }
      
      if (this.clipboardContent) {
        TextInput({ text: this.clipboardContent })
          .width('100%')
          .height(100)
          .borderRadius(8)
          .padding(8)
          .backgroundColor('#F5F5F5')
          .margin({ top: 12 })
      } else {
        Text('剪贴板为空')
          .fontSize(14)
          .fontColor('#CCCCCC')
          .padding(12)
          .width('100%')
          .height(100)
          .backgroundColor('#F5F5F5')
      }
      
      Divider()
      
      Row() {
        Button('复制文本到剪贴板')
          .onClick(async () => {
            const text = 'Hello from HarmonyOS!'
            await DistributedClipboardManager.setText(text)
            await this.updateClipboardContent()
          })
        
        Button('清除剪贴板')
          .onClick(async () => {
            await DistributedClipboardManager.setText('')
            await this.updateClipboardContent()
          })
          .margin({ left: 12 })
      }
      .justifyContent(FlexAlign.SpaceEvenly)
    }
    .padding(20)
  }
  
  private formatTime(date: Date): string {
    return new Date(date).toLocaleString()
  }
}
```

## 设备发现与选择

### 设备列表管理

```typescript
import deviceManager from '@ohos.distributedHardware.deviceManager'

class DeviceDiscoveryManager {
  private static deviceManager: deviceManager.DeviceManager | null = null
  private static deviceList: deviceManager.DeviceInfo[] = []
  private static onlineDevices: deviceManager.DeviceInfo[] = []
  
  static async initialize(): Promise<void> {
    try {
      this.deviceManager = deviceManager.createDeviceManager(getContext(this))
      
      // 获取设备列表
      this.deviceList = await this.deviceManager.getTrustedDeviceList()
      this.onlineDevices = this.deviceList.filter(
        device => device.deviceState === deviceManager.DeviceState.STATE_ONLINE
      )
      
      // 监听设备变化
      this.setupDeviceListeners()
      
      console.log('设备发现管理器初始化完成')
      console.log('总设备数:', this.deviceList.length)
      console.log('在线设备数:', this.onlineDevices.length)
    } catch (error) {
      console.error('初始化设备发现管理器失败:', error)
      throw error
    }
  }
  
  private static setupDeviceListeners(): void {
    if (!this.deviceManager) return
    
    // 设备状态变化监听
    this.deviceManager.on('deviceStateChange', (data) => {
      console.log('设备状态变化:', data)
      this.refreshDeviceList()
    })
    
    // 设备发现监听
    this.deviceManager.on('deviceFound', (data) => {
      console.log('发现新设备:', data)
      this.refreshDeviceList()
    })
  }
  
  static async refreshDeviceList(): Promise<void> {
    try {
      this.deviceList = await this.deviceManager.getTrustedDevice()
      this.onlineDevices = this.deviceList.filter(
        device => device.deviceState === deviceManager.DeviceState.STATE_ONLINE
      )
      
      console.log('设备列表已刷新')
    } catch (error) {
      console.error('刷新设备列表失败:', error)
    }
  }
  
  static getOnlineDevices(): deviceManager.DeviceInfo[] {
    return this.onlineDevices
  }
  
  static getDeviceById(deviceId: string): deviceManager.DeviceInfo | undefined {
    return this.onlineDevices.find(device => device.deviceId === deviceId)
  }
  
  static getDeviceByName(deviceName: string): deviceManager.DeviceInfo | undefined {
    return this.onlineDevices.find(device => 
      device.deviceName === deviceName
    )
  }
  
  static getDevicesByType(deviceType: deviceManager.DeviceType): deviceManager.DeviceInfo[] {
    return this.onlineDevices.filter(device => 
      device.deviceType === deviceType
    )
  }
}

// 使用示例
@Entry
@Component
struct DeviceListPage {
  @State devices: deviceManager.DeviceInfo[] = []
  
  async aboutToAppear() {
    await DeviceDiscoveryManager.initialize()
    await this.refreshDevices()
  }
  
  async refreshDevices(): Promise<void> {
    await DeviceDiscoveryManager.refreshDeviceList()
    this.devices = DeviceDiscoveryManager.getOnlineDevices()
  }
  
  build() {
    Column() {
      Row() {
        Text('设备列表')
          .fontSize(20)
          .fontWeight(FontBold)
          .margin({ right: 12 })
        
        Button('刷新设备')
          .onClick(() => {
            this.refreshDevices()
          })
      }
      .width('100%')
      .justifyContent(FlexAlign.SpaceBetween)
      
      Divider()
      
      if (this.devices.length === 0) {
        Text('暂无在线设备')
          .fontSize(16)
          .fontColor('#999999')
          .padding(20)
          .width('100%')
          .height('200')
          .textAlign(TextAlign.Center)
      } else {
        List() {
          ForEach(this.devices, (device: deviceManager.DeviceInfo) => {
            ListItem() {
              DeviceCard({ device: device })
            }
          }, (device: deviceManager.DeviceInfo) => device.deviceId)
        }
      }
    }
    .padding(20)
  }
}

@Component
struct DeviceCard {
  @Prop device: deviceManager.DeviceInfo
  
  build() {
    Row() {
      Column() {
        Text('📱')
          .fontSize(24)
          .margin({ right: 12 })
        
        Column() {
          Text(this.device.deviceName || '未知设备')
            .fontSize(16)
            .fontWeight(FontWeight.Bold)
          
          Text(this.device.deviceType || '未知类型')
            .fontSize(14)
            .fontColor('#666666')
            .margin({ top: 4 })
          
          if (this.device.deviceId) {
            Text(`设备ID: ${this.device.deviceId}`)
              .fontSize(12)
              .fontColor('#999999')
              .margin({ top: 2 })
          }
        }
        .alignItems(VerticalAlign.Center)
      }
      .width('100%')
      .padding(16)
      .backgroundColor('#FFFFFF')
      .borderRadius(12)
      .shadow({ radius: 4, color: '#00000010' })
      .onClick(() => {
        this.showDeviceOptions()
      })
    }
  }
  
  private showDeviceOptions(): void {
    // 显示设备操作选项
    console.log('显示设备选项:', this.device.deviceName)
  }
}
```

## 最佳实践

### 1. 权限配置

```typescript
// 在module.json5中声明必要的权限
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.DISTRIBUTED_DATASYNC",
        "reason": "$string:permission_distributed_data_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.INTERACT_ACROSS_DEVICES",
        "reason": "$string:permission_cross_device_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.READ_WRITE_DOWNLOAD",
        "reason": "$string:permission_download_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

### 2. 数据安全与隐私

```typescript
// 文件分享安全检查
class SecurityValidator {
  static validateFileShare(fileInfo: FileInfo): boolean {
    // 检查文件类型
    const allowedTypes = [
      'image/jpeg',
      'image/png',
      'application/pdf',
      'text/plain',
      'application/vnd.ms-word'
    ]
    
    if (!allowedTypes.includes(fileInfo.type)) {
      console.error('不支持的文件类型:', fileInfo.type)
      return false
    }
    
    // 检查文件大小
    const maxSize = 100 * 1024 * 1024 // 100MB
    if (fileInfo.size > maxSize) {
      console.error('文件过大:', fileInfo.size, 'bytes')
      return false
    }
    
    return true
  }
  
  static validateFileDownload(fileInfo: FileInfo): boolean {
    // 检查下载来源的合法性
    if (!fileInfo.sourceDeviceId) {
      return false
    }
    
    // 检查是否过期
    if (fileInfo.expiresAt < Date.now()) {
      return false
    }
    
    return true
  }
}
```

### 3. 性能优化

```typescript
// 批量文件操作优化
class FileOperationOptimizer {
  private static batchQueue: FileOperation[] = []
  private static batchSize: number = 5
  
  static async processInBatches<T>(
    operations: FileOperation<T>[],
    processFn: (batch: FileOperation<T>[]) => Promise<T[]>
  ): Promise<T[]> {
    const results: T[] = []
    
    for (let i = 0; i < operations.length; i += this.batchSize) {
      const batch = operations.slice(i, i + this.batchSize)
      const batchResults = await processFn(batch)
      results.push(...batchResults)
      
      // 给系统时间处理
      await new Promise(resolve => setTimeout(resolve, 50))
    }
    
    return results
  }
}

interface FileOperation<T> {
  operation: () => Promise<T>
  fileInfo: FileInfo
  priority: number
}

// 使用示例
const fileOperations: FileOperation[] = [
  {
    operation: () => uploadFile(file1),
    fileInfo: file1,
    priority: 1
  },
  {
    operation: () => uploadFile(file2),
    fileInfo: file2,
    priority: 2
  },
  {
    operation: () => uploadFile(file3),
    fileInfo: file3,
    priority: 3
  }
]

const results = await FileOperationOptimizer.processInBatches(
  fileOperations,
  async (batch) => {
    const batchResults = await Promise.all(
      batch.map(op => op.operation())
    )
    return batchResults
  }
)
```

### 4. 用户体验优化

```typescript
// 文件传输进度反馈
class FileTransferProgress {
  private static transfers: Map<string, TransferProgress> = new Map()
  
  static startTransfer(fileId: string, totalSize: number): void {
    const progress: TransferProgress = {
      fileId: fileId,
      transferredSize: 0,
      totalSize: totalSize,
      status: 'pending',
      startTime: Date.now()
    }
    
    this.transfers.set(fileId, progress)
    console.log(`开始传输文件 ${fileId}`)
  }
  
  static updateProgress(fileId: string, transferredSize: number, status: string): void {
    const progress = this.transfers.get(fileId)
    if (!progress) return
    
    progress.transferredSize = transferredSize
    progress.status = status
    
    const percentage = (transferredSize / progress.totalSize) * 100
    console.log(`文件 ${fileId} 传输进度: ${percentage.toFixed(2)}%`)
  }
  
  static getProgress(fileId: string): TransferProgress | undefined {
    return this.transfers.get(fileId)
  }
  
  static completeTransfer(fileId: string): void {
    const progress = this.transfers.get(fileId)
    if (!progress) return
    
    progress.status = 'completed'
    const duration = Date.now() - progress.startTime
    
    console.log(`文件 ${fileId} 传输完成，耗时: ${duration}ms`)
  }
}

interface TransferProgress {
  fileId: string
  transferredSize: number
  totalSize: number
  status: 'pending' | 'transferring' | 'completed' | 'failed'
  startTime: number
}
```

## 参考

- [华为官方文档 - 文件分享指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-file-share-download)
- [华为官方文档 - 拖拽开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/use-pasteboard-to-copy-and-paste)
- [华为官方文档 - 统一拖拽](https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-unified-drag-and-drop)
- [华为官方文档 - 跨端拖拽](https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-distribute-drag-cast)
- [华为官方文档 - 跨端文件分享](https://developer.huawei.com/consumer/cn/doc/best-practices/bpta-application-knock-file-share)
