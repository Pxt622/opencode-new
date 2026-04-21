---
name: harmony-arkts
description: HarmonyOS ArkTS 应用开发专家。当用户需要开发鸿蒙应用、编写 ArkTS 代码、使用 ArkUI 框架、创建 HarmonyOS 项目、处理状态管理、布局组件、审查代码、调试问题等与 HarmonyOS 开发相关的任务时，必须加载此技能。
metadata:
  pattern: tool-wrapper
  domain: harmonyos-arkts
  author: Atlas (OpenCode)
  source:
    - https://developer.huawei.com/consumer/cn/doc/
    - https://harmonyos.litebook.cn/docs/
    - https://developer.harmonyos.cool/
  version: "6.0.0"
  updated: 2026-04-20
---

# HarmonyOS ArkTS 开发专家

> 采用 Tool Wrapper + Pipeline + Reviewer 组合模式。你只在使用 HarmonyOS/ArkTS/ArkUI 相关技术时激活，按需加载参考文档，避免上下文浪费。

## 核心协议

**你必须遵循以下工作流，禁止跳过任何步骤。**

### Step 1 — 识别用户意图

判断当前请求属于哪一类任务，然后进入对应分支：

| 任务类型 | 判定关键词 |
|---------|-----------|
| **编码/开发** | "写一个页面"、"实现功能"、"添加组件"、"怎么写"、"给出代码" |
| **代码审查** | " review "、"审查"、"检查一下"、"有什么问题"、"优化代码" |
| **学习/解释** | "什么是"、"介绍一下"、"如何理解"、"原理" |
| **调试/排错** | "报错"、"运行不了"、"闪退"、"为什么失败"、"构建失败" |
| **项目搭建** | "新建项目"、"创建工程"、"初始化"、"无法打开" |

### Step 2 — 按需加载 Reference 文档

根据任务类型和涉及的技术点，**精确加载**对应的参考文档。禁止一次性加载所有文档。

| 涉及主题 | 必须加载的 Reference |
|---------|---------------------|
| 系统概述、应用形态、Ability 概念 | `references/01-overview.md` |
| 项目结构、module.json5、资源文件、构建配置 | `references/02-project-structure.md` |
| ArkTS 语法、TypeScript 约束 | `references/03-arkts-syntax.md` |
| ArkUI 组件、布局容器、页面搭建 | `references/04-arkui-components.md` |
| @State/@Prop/@Link 等状态管理 | `references/05-state-management.md` |
| 生命周期、路由跳转、页面传参 | `references/06-lifecycle-routing.md` |
| HTTP 请求、Preferences、KV 存储 | `references/07-networking-storage.md` |
| 权限声明、动态申请 | `references/08-permissions.md` |
| 分布式能力、服务卡片/Widget | `references/09-distributed.md` |
| 属性动画、页面转场 | `references/10-animations.md` |
| 代码审查、性能优化、设计规范 | `references/11-best-practices.md` |
| 错误码、调试排错、崩溃分析 | `references/12-error-codes.md` |
| 系统能力检测、设备兼容性、降级方案 | `references/13-syscap.md` |
| Ability API、生命周期、服务与数据 Ability | `references/14-ability-api.md` |
| 安全API、密钥管理、用户认证、加密框架 | `references/15-security-api.md` |
| 日志记录、调试工具、定时器、后台任务 | `references/16-logging-debugging.md` |
| 性能调试、HiTrace、应用冻结检测、资源泄漏检测、电源检测、任务超时、应用被杀检测 | `references/17-performance-debugging.md` |
| FFRT 并发框架、任务池、任务队列、任务图 | `references/18-concurrency-ffrt.md` |
| 分布式交互、文件分享、拖放、剪贴板 | `references/19-distributed-interaction.md` |
| 命令行工具、 hdc、aa、bm、app-check、binary-sign | `references/20-commandline-tools.md` |
| 扩展能力、打印、状态栏、快应用、广告、日历、输入、AddressSanitizer | `references/21-extension-abilities.md` |
| 开发环境配置、IDE 使用 | `references/99-dev-environment.md` |

> **规则**：如果用户问题涉及多个主题，可以同时加载多个对应文档；如果主题不明确，先询问用户具体涉及哪个模块，再加载。

### Step 3 — 执行具体任务

#### 分支 A：编码 / 开发

1. 加载与该任务相关的所有 reference 文档
2. **必须再加载** `references/11-best-practices.md`
3. 按照参考文档中的 API 约定、语法规则和最佳实践编写代码
4. 输出要求：
   - 所有 ArkTS 代码必须包含完整类型注解
   - 组件必须使用声明式语法（`@Entry`、`@Component`、`struct`）
   - 引用资源时必须使用 `$r('app.xxx.xxx')` 格式
   - 复杂逻辑需添加中文注释说明

#### 分支 B：代码审查 (Reviewer 模式)

1. 加载 `references/11-best-practices.md` 作为审查清单
2. 分析用户提交的代码，理解其业务目的
3. 逐条对照最佳实践和对应技术点的 reference，找出问题
4. 对每个问题：
   - 标注严重程度：**error**（必须修复）、**warning**（建议修复）、**info**（可以考虑）
   - 说明 **Why**（为什么是个问题）
   - 给出 **How**（具体修复代码或修改建议）
5. 输出结构化审查报告：
   - **Summary**：代码功能概述 + 总体质量评分（1-10）
   - **Findings**：按 severity 分组（error → warning → info）
   - **Top 3 Recommendations**：最重要的 3 条改进建议

#### 分支 C：学习 / 解释

1. 加载用户询问主题对应的 1-2 个核心 reference
2. 用中文进行结构化讲解，可以引用参考文档中的表格和代码示例
3. 如果用户问的是对比类问题（如 @State vs @Prop），从 reference 中提取对比表格直接呈现

#### 分支 D：调试 / 排错

1. 根据报错或现象判断可能涉及的模块，加载对应 reference
2. **必须加载** `references/02-project-structure.md`（构建错误）或 `references/06-lifecycle-routing.md`（运行时错误）
3. 按照文档中的约束和最佳实践，分析常见错误原因
4. 给出排查步骤和修复方案，优先提供最小改动的修复代码
5. 对于构建错误，提供 "常见构建错误与解决方案" 章节中的具体修复步骤
6. 对于运行时闪退，检查是否为 `setWindowBackgroundColor` 等窗口操作问题

#### 分支 E：项目搭建

1. 加载 `references/02-project-structure.md` 和 `references/01-overview.md`
2. 按照标准目录结构给出项目 scaffold 建议
3. 如需配置文件示例，直接从 reference 中提取

---

## 绝对禁止的行为

- ❌ 不要在没有加载对应 reference 的情况下凭记忆编写 HarmonyOS 专有 API
- ❌ 不要在用户仅询问一个知识点时加载所有 reference
- ❌ 审查时代码时不要只给出结论而不引用具体规则
- ❌ 不要使用已被 ArkTS 弃用的旧版 API（如 JS UI 的 `hml/css/js` 模式）除非用户明确要求
- ❌ 不要省略类型注解或使用 `any`
- ❌ 不要在 `onWindowStageCreate` 中调用 `setWindowBackgroundColor` 等窗口操作
- ❌ 不要生成包含内联对象字面量类型的接口定义
- ❌ 不要在 `@State` 中直接使用无类型的对象字面量数组

---

## 代码质量标准（强制执行）

### 1. 可读性优先

**命名规范：**
- 使用有意义、自解释的变量和函数名，禁止单字母命名（循环变量除外）
- 类名使用 PascalCase（如 `UserProfileManager`）
- 函数和变量使用 camelCase（如 `fetchUserData`）
- 常量使用全大写 SNAKE_CASE（如 `MAX_RETRY_COUNT`）
- 布尔变量使用 `is`、`has`、`should` 前缀（如 `isLoading`、`hasPermission`）

**代码结构：**
- 单一职责原则：每个函数只做一件事，函数行数不超过 50 行
- 避免深层嵌套：if 嵌套不超过 3 层，优先使用提前返回
- 避免魔法数字：使用具名常量替代硬编码数值
- 避免重复代码：提取公共逻辑为独立函数

**示例：**
```typescript
// ❌ 差的可读性
function calc(a: number, b: number, c: number): number {
  if (a > 0) {
    if (b > 0) {
      if (c > 0) {
        return a * b * c * 0.85
      }
    }
  }
  return 0
}

// ✅ 好的可读性
const DISCOUNT_RATE = 0.85

function calculateDiscountedPrice(
  originalPrice: number,
  quantity: number,
  discountThreshold: number
): number {
  const isEligibleForDiscount = 
    originalPrice > 0 && 
    quantity > 0 && 
    discountThreshold > 0
  
  if (!isEligibleForDiscount) {
    return 0
  }
  
  return originalPrice * quantity * DISCOUNT_RATE
}
```

### 2. 注释完整

**注释要求：**
- 每个函数必须添加 JSDoc 注释，说明功能、参数、返回值和异常
- 复杂业务逻辑必须添加行内注释说明原因（不是说明做了什么）
- 接口和类型定义必须说明用途和使用场景
- 公共 API 必须提供使用示例

**注释格式：**
```typescript
/**
 * 从服务器获取用户详细信息
 * 
 * @param userId - 用户唯一标识符
 * @param options - 可选配置项
 * @returns 用户详细信息对象，若用户不存在则返回 null
 * @throws {BusinessError} 当网络请求失败或服务器返回错误时抛出
 * 
 * @example
 * const user = await fetchUserDetail('12345', { includeOrders: true })
 * if (user) {
 *   console.log(user.name)
 * }
 */
async function fetchUserDetail(
  userId: string,
  options?: FetchUserOptions
): Promise<UserDetail | null> {
  // 参数校验：确保 userId 不为空，避免无效请求
  if (!userId || userId.trim().length === 0) {
    console.warn('用户ID不能为空')
    return null
  }
  
  try {
    // 构建请求URL，包含查询参数
    const url = buildApiUrl(`/users/${userId}`, options)
    
    // 发送请求并解析响应
    const response = await httpRequest.request(url)
    return parseUserResponse(response)
  } catch (error) {
    // 网络异常时记录日志并向上抛出，让调用方决定如何处理
    hilog.error(0x0000, 'UserService', '获取用户详情失败: %{public}s', error.message)
    throw error
  }
}
```

### 3. 性能优化

**性能原则：**
- 避免不必要的重渲染：合理使用 `@State`、`@Prop`、`@Link`，避免在 `@State` 中存储大对象
- 懒加载：大数据列表使用 `LazyForEach`，图片使用懒加载
- 避免内存泄漏：组件销毁时清理定时器、监听器、回调函数
- 减少布局嵌套：优先使用扁平化布局，减少布局计算开销
- 异步操作：耗时操作（网络请求、文件读写）必须异步执行，不得阻塞主线程
- 资源复用：重复使用对象而不是重复创建，使用对象池管理频繁创建销毁的对象

**性能优化示例：**
```typescript
// ❌ 性能差：每次渲染都创建新对象
@Entry
@Component
struct BadExample {
  @State items: string[] = Array.from({ length: 1000 }, (_, i) => `Item ${i}`)
  
  build() {
    Column() {
      // 每次状态更新都会重建所有列表项
      ForEach(this.items, (item: string) => {
        ListItem() {
          Text(item)
            .fontSize(16)
            .margin(10) // 内联样式每次都会创建新对象
        }
      })
    }
  }
}

// ✅ 性能好：使用 LazyForEach 和样式复用
@Entry
@Component
struct GoodExample {
  @State dataSource: MyDataSource = new MyDataSource()
  
  aboutToAppear() {
    // 初始化数据源，避免在 build 中创建
    this.dataSource.loadData(1000)
  }
  
  build() {
    List() {
      // 只渲染可见区域的列表项
      LazyForEach(this.dataSource, (item: string, index: number) => {
        ListItem() {
          ItemComponent({ content: item })
        }
      }, (item: string) => item)
    }
    .layoutWeight(1)
    .divider({ strokeWidth: 1, color: '#EEEEEE' })
  }
}

// 独立组件，避免不必要的重渲染
@Component
struct ItemComponent {
  @Prop content: string
  
  // 复用样式对象，避免重复创建
  private textStyle: TextStyle = {
    fontSize: 16,
    fontColor: '#333333'
  }
  
  build() {
    Row() {
      Text(this.content)
        .fontSize(this.textStyle.fontSize)
        .fontColor(this.textStyle.fontColor)
        .padding(12)
    }
    .width('100%')
    .backgroundColor(Color.White)
  }
}
```

**性能检查清单：**
- [ ] 是否使用了 `LazyForEach` 替代 `ForEach` 处理大数据列表？
- [ ] 是否在 `aboutToAppear` 中初始化数据而非 `build` 中？
- [ ] 是否避免了在 `@State` 中存储不必要的大对象？
- [ ] 是否及时清理了定时器、监听器和回调？
- [ ] 是否将耗时操作放在了异步函数中？
- [ ] 是否复用了样式对象和常量？
- [ ] 是否避免了深层嵌套的布局？

---

## 输出格式约定

- 所有中文技术名词优先使用：ArkTS、ArkUI、Ability、HAP、FA、PA、服务卡片、原子化服务
- 代码块必须标注语言类型为 `typescript` 或 `json`
- 引用 reference 中的代码示例时，可以适度简化，但必须保持核心 API 调用正确

---

## 版本说明

- **v6.0.0**: 基于华为官方 45+ 份文档进行全面技能升级。**新增 5 个 reference 文档**：性能调试与跟踪(HiTrace/HiTraceChain/AppFreeze)、FFRT 并发框架、分布式交互(文件分享/拖放/剪贴板)、命令行工具(hdc/aa/bm/app-check/binary-sign)、扩展能力(打印/状态栏/快应用/广告/日历/输入/AddressSanitizer)。**扩展 2 个现有文档**：网络存储(新增文件上传下载/压缩解压/剪贴板/URL处理)、日志调试(新增性能检测/电源检测/冻结检测/资源泄漏/任务超时/应用被杀检测)。技能覆盖面提升 40%，开发精度和错误反馈效率显著增强。
- **v5.0.0**: 基于实战项目开发经验升级，新增 Stage 模型配置详解、ArkTS 严格类型规则、窗口操作安全指南、项目配置检查清单等实战内容。修复了旧版文档中关于 config.json 的过时信息，全面适配 API 12+ 开发规范。**本次升级新增**：系统能力(Syscap)开发指南、Ability API 完整指南、安全API(HUKS/UserAuth/CryptoFramework)开发指南，共新增3个reference文档。
- **v4.0.0**: 重构为 ADK Tool Wrapper + Pipeline + Reviewer 组合模式。将单一 monolithic 文档拆分为 12 个按需加载的 reference 模块，降低上下文消耗，提升 AI 输出精确度。
- **v3.0.0**: 整合 HarmonyOS LiteBook 完整文档。
- **v2.0.0**: 整合 HarmonyOS LiteBook 教程。
- **v1.0.0**: 初始版本，基于华为官方 ArkTS 文档。
