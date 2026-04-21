# HarmonyOS 扩展能力完整指南

## 扩展能力概览

HarmonyOS 提供了丰富的扩展能力，让应用能够深度集成系统服务、创建沉浸式用户体验。这些能力通过特定的API和配置实现。

| 扩展类型 | 主要API | 适用场景 |
|---------|--------|---------|
| **打印扩展** | `@ohos.print` | PDF打印、标签打印、图片打印 |
| **状态栏扩展** | `@ohos.statusbar` | 系统状态展示、消息通知 |
| **快捷栏扩展** | `@ohos.quickbar` | 桌面快捷方式、系统级快捷操作 |
| **LiveView** | `@ohos.liveview` | 桌面动态卡片、实况直播、虚拟人 |
| **广告服务** | `@ohos.ads` | 广告集成、激励视频、开屏广告 |
| **日历服务** | `@ohos.calendar` | 日历事件管理、日程提醒、日历访问 |

## 一、PrintExtensionAbility（打印扩展）

### 基础概念

打印扩展是Ability的特殊类型，用于与打印相关服务进行交互，主要功能包括：

- 打印机发现
- 打印任务管理
- 打印数据获取
- 打印状态监听
- 错误处理

### 创建打印扩展Ability

```typescript
import print from '@ohos.print'
import print from '@ohos.print.Printer'

@Entry
@Component
struct MyPrintExtension {
  @State printerId: string = ''
  @State connected: boolean = false
  @State printerName: string = ''
  
  aboutToAppear() {
    this.startPrintService()
  }
  
  build() {
    Column() {
      Text('打印扩展示例')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      
      if (this.connected) {
        Text(`已连接打印机: ${this.printerName}`)
          .fontColor('#00AA00')
      } else {
        Text('未连接打印机')
          .fontColor('#FF0000')
      }
      
      Button('连接打印机')
        .onClick(() => this.connectPrinter())
      
      if (this.connected) {
        Button('打印测试')
          .onClick(() => this.printTestPage())
          .margin({ top: 20 })
        
        Button('打印图片')
          .onClick(() => this.printImage())
          .margin({ top: 12 })
      }
      .width('100%')
      .padding(20)
  }
  
  // 连接打印机
  private async connectPrinter(): Promise<void> {
    try {
      // 获取打印机列表
      const printers = await print.getPrinters()
      
      if (printers.length === 0) {
        console.log('未发现打印机')
        return
      }
      
      console.log('发现打印机:', printers)
      
      // 选择第一台打印机
      const printer = printers[0]
      this.printerId = printer.printerId
      this.printerName = printer.printerName
      
      // 连接打印机
      await printer.connect()
      this.connected = true
      
      console.log('打印机连接成功:', this.printerName)
    } catch (error) {
      console.error('连接打印机失败:', error)
      this.connected = false
      this.printerName = ''
    }
  
  // 打印测试页
  private printTestPage(): void {
    const page: print.PrintDocument = {
      data: 'Hello from HarmonyOS!',
      type: print.PrintType.PLAIN
    }
    
    print.print(this.printerId, page, (err) => {
      if (err) {
        console.error('打印失败:', err)
        return
      }
      
      console.log('打印完成')
    })
  }
  
  // 打印图片
  private printImage(): void {
    const imagePath = '/data/local/images/print.jpg'
    const image: print.PixelMap = {
      src: `file://${imagePath}`
    }
    
    const page: print.PrintDocument = {
      data: {
        type: print.PrintType.IMAGE,
        content: 'HarmonyOS图片打印示例'
      },
      type: print.PrintType.IMAGE
    }
    
    print.print(this.printerId, page, (err) => {
      if (err) {
        console.error('打印图片失败:', err)
        return
      }
      
      console.log('图片打印完成')
    })
  }
}
```

### 打印数据获取

```typescript
class PrintDataManager {
  private static async fetchPrintData(
    printerId: string
  ): Promise<PrintData> {
    try {
      const printData = await print.getPrintData(printerId)
      
      console.log('打印数据:', printData)
      
      return {
        printerId: printData.printerId,
        printerName: printData.printerName || '',
        capability: printData.capability,
        printerType: printData.printerType,
        status: printData.printerStatus
      }
    } catch (error) {
      console.error('获取打印数据失败:', error)
      throw error
    }
  }
}

interface PrintData {
  printerId: string
  printerName: string
  capability: any
  printerType: string
  status: print.PrinterStatus
}
```

## 二、StatusBarExtensionAbility（状态栏扩展）

### 基础概念

状态栏扩展允许在系统状态栏（顶部或底部）显示应用的自定义内容，支持多种UI组件和交互。

### 创建状态栏扩展

```typescript
import statusBar from '@ohos.statusbar';
import notificationManager from '@ohos.notificationManager'

@Entry
@Component
struct MyStatusBarExtension {
  @State content: string = '来自状态栏扩展的消息'
  
  async aboutToAppear() {
    await this.initStatusBar()
  }
  
  build() {
    Column() {
      Text('状态栏扩展示例')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      
      Text(this.content)
        .fontSize(16)
        .fontColor('#666666')
      
      Button('更新状态栏')
        .onClick(() => this.updateStatusBarContent())
        .margin({ top: 20 })
    }
    .width('100%')
    .padding(20)
  }
  
  // 初始化状态栏
  private async initStatusBar(): Promise<void> {
    try {
      const statusBar = statusBar.createStatusBar()
      
      // 配置状态栏
      const statusBarConfig: statusBar.StatusConfig = {
        type: statusBar.Type.EXTENSION,
        height: 60,
        backgroundType: statusBar.BackgroundType.DEFAULT,
        show: true
      }
      
      await statusBar.setStatusBarConfig(statusBarConfig)
      
      // 设置初始内容
      await this.updateStatusBarContent()
      
      console.log('状态栏扩展初始化成功')
    } catch (error) {
      console.error('状态栏扩展初始化失败:', error)
      throw error
    }
  }
  
  // 更新状态栏内容
  private async updateStatusBarContent(): Promise<void> {
    try {
      const statusBar = statusBar.get()
      
      // 更新文本内容
      const textContent: statusBar.TextContent = {
        text: this.content,
        isShow: true
      }
      await statusBar.setContents(textContent)
      
      console.log('状态栏内容已更新')
    } catch (error) {
      console.error('更新状态栏失败:', error)
      }
  }
  
  // 显示通知
  private async showNotification(): Promise<void> {
    try {
      await this.updateStatusBarContent()
      
      // 显示通知
      const notification = notificationManager.createNotification()
      notification.show({ title: '状态栏通知', content: '这是状态栏通知' })
      
    } catch (error) {
      console.error('显示通知失败:', error)
    }
  }
}
```

### 状态栏UI组件

```typescript
@Component
struct StatusBarCard {
  @Builder
  static buildRow(options: { title: string, action?: () => void }) {
    Row() {
      Text(options.title)
        .fontSize(16)
        .fontColor('#333333')
        .margin({ right: 16 })
      
      if (options.action) {
        Button('执行')
          .onClick(options.action)
          .fontSize(14)
          .fontColor('#007AFF')
      }
    }
  }
}
```

### 最佳实践

1. **高度适配**：根据设备形态调整状态栏高度（手机35-60，平板60-100）
2. **内容更新频率**：避免频繁更新，建议使用定时器批量更新
3. **内存管理**：及时释放状态栏对象，避免内存泄漏
4. **异常处理**：连接/更新失败时提供降级方案
5. **生命周期**：在aboutToDisappear中释放资源，取消监听

## 三、QuickBarExtensionAbility（快捷栏扩展）

### 基础概念

快捷栏扩展允许在设备桌面或系统界面的固定位置显示应用快捷方式，支持点击打开特定功能或网页链接。

### 创建快捷栏扩展

```typescript
import quickbar from '@ohos.quickbar'
import router from '@ohos.router'
import { BusinessAbility } from '@ohos.ability.BusinessAbility'

@Entry
@Component
struct MyQuickBarExtension {
  @State shortcutId: string = ''
  @State shortcutName: string = ''
  
  async aboutToAppear() {
    await this.initQuickBar()
  }
  
  build() {
    Column() {
      Text('快捷栏扩展示例')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      
      Text(`当前快捷方式: ${this.shortcutName}`)
        .fontSize(16)
        .fontColor('#666666')
      
      Button('添加快捷方式')
        .onClick(() => this.showAddShortcutDialog())
        .margin({ top: 20 })
      
      Divider()
      
      List() {
        ListItem({ title: '打开首页', action: () => this.openHome() })
        ListItem({ title: '打开详情', action: () => this.openDetail() })
        ListItem({ title: '打开设置', action: () => openSettings() })
      }
      .width('100%')
      .padding(20)
    }
  }
  
  // 初始化快捷栏
  private async initQuickBar(): Promise<void> {
    try {
      const quickBar = quickbar.createQuickBar()
      
      // 创建快捷方式
      const shortcut = {
        id: 'open_home',
        shortcutId: 'my_app_open_home',
        appName: 'com.example.myapp',
        abilityName: 'EntryAbility',
        icon: $media:icon',
        title: '打开应用',
        type: 'universal' // 通用快捷方式
      }
      
      await quickBar.addQuickBar(shortcut)
      
      // 获取已存在的快捷方式
      const shortcuts = await quickBar.getQuickBars()
      this.shortcutId = shortcuts[0]?.shortcutId || ''
      this.shortcutName = shortcuts[0]?.title || '打开应用'
      
      console.log('快捷栏初始化完成')
    } catch (error) {
      console.error('快捷栏初始化失败:', error)
    }
  }
  
  // 添加新快捷方式
  private async showAddShortcutDialog(): Promise<void> {
    try {
      await this.showAddShortcutDialog({
        title: '添加快捷方式',
        content: this.getShortcutDialog()
      })
    } catch (error) {
      console.error('显示添加快捷方式对话框失败:', error)
    }
  }
  
  // 打开指定页面
  private openPage(url: string): void {
    if (url.startsWith('http')) {
      // 打开网页链接
      router.pushUrl(url)
    } else {
      // 打开应用内页面
      router.pushUrl({ url: 'pages/Detail', params: { id: 1 } })
    }
  
  private openHome(): void {
    this.openPage('pages/Index')
  }
  
  private openDetail(): void {
    this.openPage('pages/Detail')
  }
  
  private openSettings(): void {
    openSettings()
  }
}
```

### 快捷方式管理

```typescript
class ShortcutManager {
  private static async addShortcut(
    title: string,
    url: string,
    icon?: Resource
  ): Promise<string> {
    const quickBar = quickbar.createQuickBar()
    
    const shortcut = {
      id: Date.now().toString(),
      shortcutId: 'shortcut_' + Date.now(),
      appName: 'com.example.myapp',
      abilityName: 'EntryAbility',
      icon: icon || $r('app.media.icon'),
      title: title,
      type: 'app',
      want: {
        bundleName: 'com.example.myapp',
        abilityName: 'EntryAbility',
        moduleName: 'entry',
        path: url.startsWith('http') ? url : `pages/${url}`,  // 自动识别外部链接和应用内路径
        action: {
          action: 'openUrl',
          data: url
        }
      }
    }
    
    await quickBar.addQuickBar(shortcut)
    return shortcut.id
  }
  
  static async getAllShortcuts(): Promise<any[]> {
    const quickBar = quickbar.createQuickBar()
    return await quickBar.getQuickBars()
  }
  
  static async removeShortcut(shortcutId: string): Promise<void> {
    const quickBar = quickbar.createQuickBar()
    await quickBar.removeQuickBar(shortcutId)
    console.log('快捷方式已删除')
  }
  
  static async clearAllShortcuts(): Promise<void> {
    const quickBar = quickbar.createQuickBar()
    
    const shortcuts = await quickBar.getQuickBars()
    for (const shortcut of shortcuts) {
      await quickBar.removeQuickBar(shortcut.id)
    }
    
    console.log(`已清空 ${shortcuts.length} 个快捷方式`)
  }
}
```

### 快捷方式对话框

```typescript
@Component
struct AddShortcutDialog {
  @State title: string = ''
  @State url: string = ''
  @State iconName: string = ''
  @State showDialog: boolean = false
  
  // 对话框输入
  @State inputTitle: string = ''
  @State inputUrl: string = ''
  @State iconPath: string = ''
  
  build() {
    Column() {
      Text('添加快捷方式')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      
      TextInput({
        placeholder: '输入标题',
        text: $this.inputTitle,
        onValueChange: (value: string) => {
          this.inputTitle = value
        },
        height: 50,
        backgroundColor('#F0F0F0')
      })
      
      TextInput({
        placeholder: '输入URL或页面路径',
        text: $this.inputUrl,
        onValueChange: (value: string) => {
          this.inputUrl = value
        },
        height: 50,
        backgroundColor('#F0F0F0')
      })
      
      Row() {
        Button('选择图标')
          .onClick(() => this.selectIcon())
          .fontSize(14)
          .fontColor('#007AFF')
        
        if (this.iconPath) {
          Image(this.iconPath)
            .width(24)
            .height(24)
            .margin({ left: 12 })
            .borderRadius(4)
            .backgroundColor('#F5F5F5')
        } else {
          Image($r('app.media.icon'))
            .width(24)
            .height(24)
            .margin({ left: 12 })
            .borderRadius(4)
            .backgroundColor('#DDDDDD')
        }
      }
        .width('100%')
        .justifyContent(FlexAlign.SpaceBetween)
      
      Row() {
        Button('取消')
          .onClick(() => {
            this.closeDialog()
          })
          .margin({ right:  })
        
        Button('确定')
          .onClick(() => {
            if (this.inputTitle && this.inputUrl) {
              this.addShortcut()
            }
          })
          .width('100%')
          .justifyContent(FlexAlign.SpaceBetween)
      }
      }
    }
    }
    .width('100%')
    .height('60%')
    .padding(20)
    .backgroundColor('#FFFFFF')
    .borderRadius(12)
    .visibility(this.showDialog ? Visibility.Visible : Visibility.None)
  }
  
  private selectIcon(): void {
    this.selectFile({
      'accept': ['image/jpeg', 'image/png', 'image/webp'],
      onConfirm: (uris: string) => {
        this.iconPath = uri
      }
    })
  }
  
  private selectFile(params: {
    accept: string[]
    onConfirm: (uri: string) => {
      this.iconPath = uri
    }
  }): void {
    picker.open(params)
  }
  
  private addShortcut(): void {
    this.showDialog = false
    console.log('快捷方式已添加')
  }
  
  private closeDialog(): void {
    this.showDialog = false
    this.inputTitle = ''
    this.inputUrl = ''
    this.iconPath = ''
  }
}
```

### 最佳实践

1. **图标规范**：推荐尺寸24x24或48x48，支持PNG/JPG格式，背景透明
2. **命名规范**：使用动词+名词格式（如"打开首页"、"添加到购物车"）
3. **链接处理**：外部链接需包含http://或https://，应用内路径直接使用pages/路径
4. **权限处理**：URL访问可能需要ohos.permission.INTERNET
5. **深度链接**：深度链接建议选择App Linking直达，用户体验更好
6. **状态保持**：保存持久化用户的快捷方式配置，支持云同步

## 四、LiveViewExtensionAbility（实况窗服务）

### 基础概念

LiveView是华为提供的一项内容分发能力，允许应用在桌面端展示内容卡片、实时数据等，用户可以通过LiveView卡片直接与内容进行交互。

### 创建LiveView扩展

```typescript
import liveView from '@ohos.liveview'
import { LiveViewManager, LiveViewConfig } from '@ohosliveview'

@Entry
@Component
struct MyLiveViewExtension {
  @State cards: LiveViewCard[] = []
  @State isLoading: boolean = true
  
  async aboutToAppear() {
    await this.loadLiveViewData()
  }
  
  build() {
    Column() {
      Text('LiveView扩展示例')
        .fontSize(20)
        .fontWeight(FontWeight.Bold)
      
      if (this.isLoading) {
        LoadingProgress()
      }
      
      if (this.cards.length === 0) {
        Text('暂无LiveView卡片')
          .fontSize(16)
          .fontColor('#666666')
      }
      
      ForEach(this.cards, (card: LiveViewCard) => {
        LiveViewCard({ card: card })
          .margin({ bottom: 16 })
      })
      
      Button('刷新LiveView数据')
        .onClick(() => this.refreshLiveView())
        .margin({ top: 20 })
      }
      .width('100%')
      .padding(20)
    }
  }
  
  private async loadLiveViewData(): Promise<void> {
    try {
      this.isLoading = true
      
      // 创建LiveView管理器
      const liveViewManager = LiveViewManager.createLiveViewManager(getContext(this))
      
      // 获取LiveView卡片列表
      const cards = await liveViewManager.getCards()
      this.cards = cards.map(card => ({
        type: 'card',
        title: card.title,
        content: card.content,
        actions: card.actions,
        metadata: card.metadata
      }))
      
      console.log(`加载了 ${this.cards.length} 个LiveView卡片`)
    } catch (error) {
      console.error('加载LiveView数据失败:', error)
    } finally {
      this.isLoading = false
    }
  }
  
  private async refreshLiveView(): Promise<void> {
    await this.loadLiveViewData()
  }
}
```

### LiveViewCard组件

```typescript
@Component
struct LiveViewCard {
  @Prop card: LiveViewCard
  
  build() {
    Column() {
      // 卡片标题
      Text(this.card.title)
        .fontSize(18)
        .fontColor(this.card.titleColor || '#333333')
        .fontWeight(FontWeight.Medium)
        .padding({ left: 16, right: 16, top: 8 })
      
      // 卡片内容（根据类型渲染不同UI）
      if (this.card.type === 'card') {
        // 普通卡片类型
        this.renderCard()
      } else if (this.card.type === 'image') {
        // 图片类型
        this.renderImage()
      } else if (this.card.type === 'text') {
        // 文本类型
        this.renderText()
      } else {
        // 其他类型
        this.renderOther()
      }
      
      Divider()
    }
  }
  
  private renderCard(): void {
    Column() {
      // 内容区域
      Text(this.card.content)
        .fontSize(14)
        .fontColor('#666666')
        .padding({ left: 16, right: 16, top: 8, bottom: 8 })
      
      // 底部操作区
      Row() {
        ForEach(this.card.actions, (action) => {
          Button(action.label)
            .fontSize(14)
            .margin({ left: 8 })
            .onClick(() => {
              this.executeAction(action)
            })
            .margin({ right: 8 })
        })
      }
      }
      .padding({ left: 16, right: 16, top: 8, bottom: 8 })
      .width('100%')
    }
  }
  
  private renderImage(): void {
    Image(this.card.imageUrl)
      .width('100%')
      .height(200)
    }
  
  private renderText(): void {
    Text(this.card.content)
      .fontSize(16)
      .fontColor('#666666')
      .padding({ left: 16, right: 16, top: 8, bottom: 8 })
      .width('100%')
  }
  
  private renderOther(): void {
    Text('卡片类型暂不支持')
      .fontSize(14)
      .fontColor('#999999')
      .padding(  {
        left: 16, right: 16, top: 8, bottom: 8 }
      .width('100%')
  }
}

// LiveViewCard数据结构
interface LiveViewCard {
  id: string
  title: string
  content: string
  type: 'card' | 'image' | 'text' | 'link' | 'video'
  titleColor?: string
  contentColor?: string
  imageUrl?: string
  actions: Action[]
  metadata?: any
}

interface Action {
  id: string
  label: string
  action: string
  priority: number
  uri?: string
}
```

### LiveView配置

```typescript
const liveViewConfig: LiveViewConfig = {
  name: 'MyApp LiveView',
  // 应用基础信息
  icon: $r('app.media.icon'),
  appId: 'com.example.myapp',
  version: '1.0.0',
  
  // LiveView特定配置
  enableDebug: true,
  autoUpdateInterval: 300, // 5分钟更新一次
  retryCount: 3,
  fallbackImageUrl: $r('app.media.fallback_icon'),
  
  // 卡片配置
  cardLimit: 20, // 最大卡片数量
  allowMultiWindow: true, // 是否支持多窗口
  windowScaleMode: 'fit', // fit | contain | fill
  
  // 服务端配置
  serverUrl: 'https://liveview.xxx.com/api',
  apiKey: 'your_api_key',
  region: 'cn',
  version: 'v2'
}
```

## 五、Ads Kit（广告服务）

### 基础概念

Ads Kit 提供了多种广告类型和展示方式，帮助开发者通过广告实现应用变现。

### 广告类型

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| **Banner广告** | 页面顶部或底部展示 | 电商首页、文章页 |
| **信息流广告** | 信息流内容流插入 | 社交媒体、新闻阅读 |
| **激励视频广告** | 奖励视频、插播内容 | 短斗类、游戏 |
| **开屏广告** | 应用启动开屏广告 | 工具类、效率类 |
| **原生广告** | 原生广告 | 混度信息流广告 |

### 广告服务集成

```typescript
import ads from '@ohos.ads'
import adRequest, adLoadStatus, display, adType from '@ohos.ads'
import display from '@ohos.ads'
import adsInfo from '@ohos.ads.adsInfo'

class AdsManager {
  private static adRequest: adRequest.AdRequest | null = null
  private static adDisplay: display.Display | null = null
  private static currentAd: adRequest.AdRequest | null = null
  
  static async initialize(): Promise<void> {
    try {
      // 初始化广告服务
      await ads.init(context(this)
      this.adRequest = adRequest.AdRequest()
      
      // 配置广告请求参数
      this.adRequest.adType = adType.CURRENT_VIDEO_PLAYER
      this.adRequest.adUnit = adRequest.AdUnit.CURRENT_PLAYER
      
      this.adRequest.requestType = adRequest.RequestType.LOAD
      this.adRequest.slotId = 'home_banner' // 广告位ID
      
      this.adRequest.isBidding = false
      
      // 创建广告展示对象
      this.adDisplay = display.createDisplay()
      
      console.log('广告服务初始化成功')
    } catch (error) {
      console.error('广告服务初始化失败:', error)
      throw error
    }
  }
  
  static async loadBannerAd(): Promise<void> {
    if (!this.adRequest) {
      console.warn('广告请求未初始化，先调用initialize()')
      return
    }
    
    // 重新创建广告请求（允许重新竞价）
    this.adRequest.bidding = true
    this.adRequest.isBidding = false
    
    // 加载开屏广告
    this.currentAd = await this.adRequest.loadAd(this.adDisplay)
    
    console.log('开屏广告已加载')
  }
  
  static async loadInfoStreamAd(): Promise<void> {
    // 加载信息流广告
    this.adRequest.adType = adRequest.AdType.INFO_STREAM
    await this.currentAd?.release()
    this.currentAd = await this.adRequest.loadAd(this.adDisplay)
    
    console.log('信息流广告已加载')
  }
  
  static async loadRewardedAd(): Promise<void> {
    // 加载激励视频广告
    this.adRequest.adType = adRequest.AdType.REWARD_VIDEO
    await this.currentAd?.release()
    this.currentAd = await this.adRequest.loadAd(this.adDisplay)
    
    console.log('激励视频广告已加载')
  }
  
  static exposeCurrentAd(display: display.Display): void {
    if (!this.adDisplay) {
      return
    }
    
    // 暴露广告
    this.currentAd = this.adRequest
    this.adDisplay.showAd(display, (err) => {
      if (err) {
        console.error('显示广告失败:', err)
      return
      }
      console.log('广告已显示')
    }
  
  static hideCurrentAd(): void {
    if (!this.currentAd) {
      return
    }
    
    // 隐藏广告
    this.adDisplay.hideAd(display, (err) => {
      if (err) {
        console.error('隐藏广告失败:', err)
        return
      }
      console.log('广告已隐藏')
    }
}
```

### Banner广告配置

```typescript
// Banner广告配置
const bannerConfig: ads.BannerConfig = {
  // 广告位ID
  slotId: 'home_banner',
  
  // 广告样式
  align: 'center', // center | left | right
  width: '100%', // '100%' | '80%' | '60%'
  height: '50', // 具体高度
  top: 0,
  bottom: 0,
  layout: 'responsive', // responsive | fixed
  
  // 内容设置
  bgColor: '#FFFFFF',
  textColor: '#000000',
  buttonColor: '#00A5F07',
  linkColor: '#007AFF',
  
  // 点击目标
  clickTo: 'pages/Detail',
  clickType: 'openUrl', // openUrl | router跳转 | router.pushUrl
  },
  
  // 数据源配置
  dataProvider: {
    type: 'api',  // api | h5
    provider: 'custom'  // custom | huawei | admob |  pangle | unity | umeng | inmobid
  }
}
```

### 信息流广告配置

```typescript
// 信息流广告配置
const infoStreamConfig: ads.InfoStreamConfig = {
  // 位置
  position: 'bottom',
  
  // 间距
  padding: 12,
  backgroundColor: '#F5F5F5',
  
  // 字体
  fontColor: '#666666',
  fontSize: 14,
  lineHeight: 24,
  boldColor: '#333333'
}
```

### 开屏广告配置

```typescript
// 开屏广告配置
const splashAdConfig: ads.SplashAdConfig = {
  // 展示配置
  width: 750,
  height: 1334,
  duration: 5, // 广告秒数
  
  // 目标
  packageName: 'com.example.app',
  abilityName: 'com.example.myapp.EntryAbility',
    pageType: 'home',
    targetPage: 'pages/Index',
    
  // 广告联盟
  adNetwork: 'huawei', // huawei | admob | pangle | unity | umeng | inmobid
    appId: 'your_app_id'
  }
}
```

### 最佳实践

1. **广告频率控制**：避免广告干扰用户，设置合理的展示间隔
2. **用户体验优先**：广告内容与应用风格保持一致
3. **加载优化**：按需加载广告数据，提前预加载
4. **错误处理**：广告加载失败时使用降级方案
5. **数据统计**：记录广告展示次数、点击率、转化率等指标
6. **合规性**：遵循广告联盟政策，确保内容合规

## 六、日历服务

### 基础概念

CalendarManager 提供了跨设备的日程管理能力，包括日程同步、日历事件管理、多设备协同。

### 日历服务初始化

```typescript
import calendarManager from '@ohos.calendar'
import calendarManager from '@ohos.calendar.calendarManager'

class CalendarManagerWrapper {
  private static calendarManager: calendarManager.CalendarManager | null = null
  
  static async initialize(): Promise<void> {
    try {
      // 创建日历管理器
      this.calendarManager = calendarManager.CalendarManager.init(getContext(this))
      
      console.log('日历服务初始化成功')
    } catch (error) {
      console.error('日历服务初始化失败:', error)
      throw error
    }
  }
}
```

### 日历事件管理

```typescript
async function addCalendarEvent(
  title: string,
  startTime: Date,
  endTime: Date,
  description: string,
  location?: string,
  attendees?: string[]
): Promise<string> {
  if (!CalendarManagerWrapper.calendarManager) {
    throw new Error('日历服务未初始化')
  }
  
  const event: calendarManager.CalendarEvent = {
    title: title,
    startTime: startTime.getTime(),
    endTime: endTime.getTime(),
    description: description,
    location: location || '未设置位置',
    attendees: attendees || []
  }
  
  const eventId = await CalendarManagerWrapper.calendarManager.addEvent({
    want: {
      bundleName: 'com.example.myapp',
      abilityName: 'com.example.myapp.EntryAbility'
    },
    event
  })
  
  return eventId
}

async function updateCalendarEvent(
  eventId: string,
  updates: Partial<calendarManager.CalendarEvent>
): Promise<boolean> {
  if (!CalendarManagerWrapper.calendarManager) {
    throw new Error('日历服务未初始化')
  }
  
  try {
    const result = await CalendarManagerWrapper.calendarManager.updateEvent({
      want: {
        bundleName: 'com.example.myapp',
        abilityName: 'com.example.myapp.EntryAbility'
      },
      eventId: eventId,
      updates: updates
    })
    
    console.log('日历事件更新成功:', eventId)
    return true
  } catch (error) {
    console.error('更新日历事件失败:', error)
    return false
  }
}

async function deleteCalendarEvent(eventId: string): Promise<boolean> {
  if (!CalendarManagerWrapper.calendarManager) {
    throw new Error('日历服务未初始化')
  }
  
  try {
    const result = await CalendarManagerWrapper.calendarManager.removeEvent({
      want: {
        bundleName: 'com.example.myapp',
        abilityName: 'com.example.myapp.EntryAbility'
      },
      eventId: eventId
    })
    
    console.log('日历事件删除成功:', eventId)
    return result
  } catch (error) {
    console.error('删除日历事件失败:', error)
    return false
  }
}
```

### 日历事件查询

```typescript
async function queryCalendarEvents(
  startDate: Date,
  endDate: Date,
  callback: () => void
): Promise<calendarManager.CalendarEvent[]> {
  if (!CalendarManagerWrapper.calendarManager) {
    throw new Error('日历服务未初始化')
  }
  
  const queryParams: calendarManager.QueryParams = {
    startDate: startDate,
    endDate: endDate,
    callBack: callback
  }
  
  const result = await CalendarManagerWrapper.calendarManager.query(queryParams)
  
  console.log(`查询到 ${result.length} 个日历事件`)
  
  return result
}
```

### 多设备协同

```typescript
async function shareEventToRemoteDevice(
  eventId: string,
  deviceId: string
): Promise<void> {
  if (!CalendarManagerWrapper.calendarManager) {
    throw new Error('日历服务未初始化')
  }
  
  const result = await CalendarManagerWrapper.calendarManager.shareEvent({
    want: {
      bundleName: 'com.example.myapp',
      deviceId: deviceId
    },
    eventId: eventId
  })
  
  if (result.code === 0) {
    console.log('日历事件分享成功')
  } else {
    console.error('日历事件分享失败:', result.code)
  }
}
```

## 七、权限配置与说明

### 必需权限

```json5
{
  "requestPermissions": [
    {
      "name": "ohos.permission.CALENDAR",
      "reason": "$string:permission_calendar_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.READ_CALENDAR",
      "reason": "$string:permission_calendar_read_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.WRITE_CALENDAR",
      "reason": "$string:permission_calendar_write_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.READ_DOWNLOAD",
      "reason": "$string:permission_download_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.GET_NETWORK_INFO",
      "reason": "$string:permission_network_info_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.GET_WIFI_INFO",
      "reason": "$string:permission_wifi_info_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.INTERACT_ACROSS_DEVICES",
      "reason": "$string:permission_device_state_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.DISTRIBUTED_DEVICE_STATE",
      "reason": "$string:permission_distributed_device_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    {
      "name": "ohos.permission.DISTRIBUTED_DATASYNC",
      "reason": "$string:permission_distributed_data_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    },
    },
    {
      "name": "ohos.permission.permission.DISTRIBUTED_DATA_ASYNC",
      "reason": "$string:permission_distributed_data_async_reason",
      "usedScene": {
        "abilities": ["EntryAbility"],
        "when": "inuse"
      }
    }
  ],
  "bundleName": "com.example.myapp"
}
```

### 权限说明

| 权限 | 用途 | 注意事项 |
|------|------|---------|
| READ_CALENDAR | 读取日历数据 | 日历是用户敏感数据，谨慎处理 |
| WRITE_CALENDAR | 修改日历数据 | 需要用户确认操作 |
| READ_DOWNLOAD | 下载文件到本地 | 需要文件读取权限 |
| INTERACT_ACROSS_DEVICES | 跨设备操作 | 需要设备发现权限 |
| DISTRIBUTED_DEVICE_STATE | 跨设备状态检查 | 需要设备发现权限 |
| DISTRIBUTED_DATASYNC | 异步数据交互 | 需要网络权限 |
| DISTRIBUTED_DATA_ASYNC | 跨设备异步数据操作 | 需要网络权限 |

## 八、最佳实践总结

### 1. 权限管理

```typescript
// 权限检查器
class PermissionChecker {
  static async checkPermission(permission: string): Promise<boolean> {
    try {
      const context = getContext(this)
      const result = await context.verifyAccessToken(permission)
      
      if (!result) {
        console.error(`权限 ${permission} 未授权，申请权限`)
        const result = await context.requestPermissionsFromUser([permission])
        return result[0]
      }
      
      return true
    } catch (error) {
      console.error('权限检查失败:', error)
      return false
  }
}

// 使用示例
async function checkPermissionAndProceed() {
  const permission = 'ohos.permission.WRITE_CALENDAR'
  const hasPermission = await PermissionChecker.checkPermission(permission)
  
  if (hasPermission) {
    // 执行需要权限的操作
    await CalendarManagerWrapper.initialize()
    await addCalendarEvent('日程会议', new Date(), new Date(Date.now() + 3600000), '项目进度会议')
  } else {
    // 引导用户授权权限
    console.log('权限未授权')
  }
}
```

### 2. 生命周期管理

```typescript
// 扩展能力生命周期
class ExtensionLifecycle {
  static async handleExtensionLifecycle(): Promise<void> {
    try {
      // 检查是否在主线程
      const isMainThread = process.isMain()
      
      if (!isMainThread) {
        console.log('不在主线程，跳过某些操作')
        return
      }
      
      // 执行资源清理
      ExtensionLifecycle.releaseResources()
      
    } catch (error) {
      console.error('处理扩展生命周期失败:', error)
    }
  }
  
  private static releaseResources(): void {
    // 清理监听器
    // 释放连接
    // 释放数据库连接
    // 清空缓存
  }
}
```

### 3. 用户隐私保护

```typescript
// 隐私数据处理
class PrivacyManager {
  private static logPrivacyEvent(event: string, data: any): void {
    // 记录隐私操作
    console.log(`隐私事件: ${event}`, JSON.stringify(data))
  }
  
  static async handleUserData(data: any): Promise<boolean> {
    // 检查数据敏感性
    const isSensitive = this.checkSensitivity(data)
    
    if (isSensitive) {
      // 加密存储
      const encryptedData = this.encryptData(data)
      
      await this.saveData(encryptedData)
      
      console.log('敏感数据已加密保存')
    }
  }
  
  private static checkSensitivity(data: any): boolean {
    // 检查是否包含敏感信息
    const sensitivePatterns = [
      /手机号/iPhone/iPhone|手机/iPad/iPad/邮箱/邮箱/身份证/银行卡/密码/住址等/社保号/医保号
    ]
    
    const dataStr = JSON.stringify(data)
    
    for (const pattern of sensitivePatterns) {
      if (pattern.test(dataStr)) {
        return true
      }
    }
    
    return false
  }
  
  private static encryptData(data: any): string {
    // 使用加密API
    return 'encrypted:' + btoa(JSON.stringify(data))
  }
}
```

### 4. 多设备数据同步

```typescript
// 多设备数据同步
class DataSyncManager {
  private static async syncToRemote(deviceId: string): Promise<void> {
    try {
      // 这里应该使用分布式数据同步API
      console.log('同步数据到设备:', deviceId)
      
      // 实现跨设备数据同步逻辑
      
    } catch (error) {
      console.error('数据同步失败:', error)
    }
  }
}
```

## 九、错误处理与调试

### 常见问题排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 扩展无法启动 | 权限未配置或配置错误 | 检查module.json5配置，检查权限声明 |
| 广告无法加载 | 网络问题或广告ID错误 | 检查网络连接，验证广告联盟配置 |
| 日历功能不工作 | 权限未申请或CalendarManager未初始化 | 检查权限、初始化流程是否正确 |
| 快捷栏不显示 | quickbar未正确注册 | 检查module.json5配置、确认注册信息格式 |
| LiveView卡片不显示 | LiveView Manager未初始化或数据源问题 | 检查LiveView配置、数据源连接 |
| 广告展示异常 | 广告数据格式错误或配置参数错误 | 检查广告数据结构、配置文件 |
| 广告点击无效 | 广告点击事件未正确配置或处理 | 检查onEvent处理、点击目标跳转配置 |

### 调试技巧

```typescript
// 1. 启用扩展调试模式
class ExtensionDebugger {
  static async debugExtension(extensionType: string): Promise<void> {
    const configs = {
      'printExtension': 'debugPrintExtension',
      'statusBarExtension': 'debugStatusBarExtension',
      'quickBarExtension': 'debugQuickBarExtension',
      'liveViewExtension': 'debugLiveViewExtension'
    }
    
    const config = configs[extensionType]
    if (!config) {
      console.error(`未找到扩展类型的调试配置: ${extensionType}`)
      return
    }
    
    // 这里应该调用具体的调试接口
    console.log(`调试扩展: ${config}`)
  }
}

// 2. 日志记录
class ExtensionLogger {
  static log(extensionType: string, event: string, data?: any): void {
    const timestamp = new Date().toISOString()
    console.log(`[${timestamp}] [${extensionType}] ${event}`)
    if (data) {
      console.log(`数据:`, JSON.stringify(data))
    }
  }
  
  static logError(extensionType: string, error: Error): void {
    const timestamp = new Date().toISOString()
    console.error(`[${timestamp}] [${extensionType}] ERROR: ${error.name}: ${error.message}`)
  }
}

// 3. 状态监听
class ExtensionWatcher {
  private static watchExtension(extensionType: string): void {}
  
  private static watchExtension(extensionType: string): void {
    const watcher = ExtensionWatcher.watchExtension(extensionType)
    console.log(`开始监听扩展: ${extensionType}`)
    }
  
  static stopWatching(): void {
    ExtensionWatcher.stopWatching()
    console.log('停止监听所有扩展')
  }
}

// 使用示例
ExtensionWatcher.watchExtension('printExtension')
// ... 测试打印扩展功能
ExtensionWatcher.stopWatching()
```

## 十、性能优化

### 资源管理

```typescript
// 扩展资源管理
class ExtensionResourceManager {
  private static resources: Map<string, any> = new Map()
  
  static addResource(resourceId: string, resource: any): void {
    this.resources.set(resourceId, resource)
  }
  
  static getResource(resourceId: string): any {
    return this.resources.get(resourceId)
  }
  
  static removeResource(resourceId: string): void {
    this.resources.delete(resourceId)
    console.log(`资源 ${resourceId} 已释放`)
  }
  
  static getAllResources(): Map<string, any> {
    return new Map(this.resources)
  }
}

// 使用示例
ExtensionResourceManager.addResource('printer_connection', { state: 'connected' })
const connection = ExtensionResourceManager.getResource('printer_connection')
console.log('打印机连接状态:', connection.state)
```

### 避免内存泄漏

```typescript
// 内存泄漏避免
class MemorySafeExtension {
  private static listeners: Array<() => void> = []
  private static callbacks: Array<() => void> = []
  
  static on(event: () => void): void {
    // 注册监听器
    this.listeners.push(event)
    
    return () => {
      return () => {
        const index = this.listeners.indexOf(event)
        if (index !== -1) {
          this.listeners.splice(index, 1)
        }
    }
  }
  
  static off(event: () => void): void {
    // 取消监听器
    const index = this.listeners.indexOf(event)
    if (index !== -1) {
      this.listeners.splice(index, 1)
    }
  }
  
  static addCallback(callback: () => void): void {
    this.callbacks.push(callback)
  }
  
  static executeCallbacks(): void {
    // 执行所有回调
    for (const callback of this.callbacks) {
      try {
        callback()
      } catch (error) {
        console.error('执行回调失败:', error)
      }
    }
  }
  
  static clearAll(): void {
    this.listeners = []
    this.callbacks = []
  }
}
```

### 异步任务管理

```typescript
// 异步任务控制器
class AsyncTaskController {
  private static taskQueue: Promise<any>[] = []
  
  static async queueTask<T>(
    task: () => Promise<T>,
    priority: 'high' | 'medium' | 'low' = 'medium',
    timeout: number = 5000
  ): Promise<T> {
    const taskWrapper: TaskWrapper<T> = {
    task,
    priority,
    timeout
    }
    
    return new Promise((resolve, reject) => {
      this.taskQueue.push(taskWrapper)
      
      if (this.taskQueue.length === 1) {
        this.runNext()
      }
    })
  }
  
  private static async runNext(): Promise<void> {
    if (this.taskQueue.length === 0) {
      return
    }
    
    const taskWrapper = this.taskQueue.shift()!
    let completed = false
    
    try {
      completed = await Promise.race([taskWrapper.task, this.timeout]) as any)
    } catch (error) {
      console.error('任务执行失败:', error)
    }
    
    if (!completed) {
      console.log('任务超时或失败')
      // 重试任务
      if (taskWrapper.priority === 'high') {
        setTimeout(() => this.runNext(), 1000)
      }
    }
  }
  
  static async clearAll(): Promise<void> {
    this.taskQueue = []
  }
}
```

## 参考

- [华为官方文档 - 打印扩展开发指南](https://developer.huawei.com/consumer/cn/doc/armonyos-guides/printextensionability)
- [华为官方文档 - 状态栏扩展开发指南](https://developer.huawei.com/consumer/cn/doc/armonyos-guides/statusbar-extension-guide)
- [华为官方文档 - 桌面快捷栏开发指南](https://developer.huawei.com/consumer/cn/doc/armonyos-guides/desktop-quickbar-extension-guide)
- [华为官方文档 - LiveView设计指南](https://developer.huawei.com/consumer/cn/doc/armonyos-guides/liveview-create-locally)
- [华为官方文档 - 广告服务集成指南](https://developer.huawei.com/consumer/cn/doc/armonyos-guides/ads-introduction)
- [华为官方文档 - 广告发布指南](https://developer.huawei.com/consumer/cn/doc/armonyos-guides/ads-publisher-service-banner)
- [华为官方文档 - 日历服务概览](https://developer.huawei.com/consumer/cn/doc/armonyos-guides/calendarmanager-overview)
- [华为官方文档 - 日历API参考](https://developer.huawei.com/consumer/cn/doc/armonyos-guides/calendarmanager-api)
- [华为官方文档 - 输入法扩展指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/input-overview)
- [华为官方文档 - 按键事件指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/keypressed-guidelines)
