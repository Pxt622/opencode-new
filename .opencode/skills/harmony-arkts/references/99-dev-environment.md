# HarmonyOS 开发环境与学习资源

## DevEco Studio

- **下载**: https://developer.huawei.com/consumer/cn/deveco-studio/
- **创建项目**: File → New → Project → ArkTS Template
- **运行**: USB 连接设备或启动模拟器

## 环境要求

| 要求 | 说明 |
|------|------|
| JDK | 11+ |
| Node.js | 18+ |
| SDK | HarmonyOS SDK |

## DevEco Studio 版本对应关系

| IDE 版本 | HarmonyOS 版本 | API 版本 | 说明 |
|---------|---------------|---------|------|
| 3.x | HarmonyOS 3.x | API 9 | Stage 模型引入 |
| 4.x | HarmonyOS 4.x | API 11 | ArkTS 增强 |
| 5.x | HarmonyOS NEXT | API 12 | 严格类型检查 |
| 6.x | HarmonyOS NEXT | API 12+ | 最新特性 |

> ⚠️ **注意**：DevEco Studio 6.0.x 对应 HarmonyOS NEXT，使用 API 12+，配置格式与旧版不同。

## 学习资源

| 资源 | 链接 |
|------|------|
| 华为官方文档 | https://developer.huawei.com/consumer/cn/doc/ |
| HarmonyOS Developer | https://developer.harmonyos.cool/ |
| HarmonyOS LiteBook | https://harmonyos.litebook.cn/docs/ |
| DevEco Studio | https://developer.huawei.com/consumer/cn/deveco-studio/ |
| GitHub 示例 | https://github.com/Awesome-HarmonyOS/HarmonyOS |
| 官方示例 | https://gitcode.com/HarmonyOS_Samples/ |

## 关键文档链接

### 项目配置
- [应用配置文件](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-hvigor-build-profile-app)
- [Stage 模型开发概述](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/stage-model-development-overview)
- [module.json5 配置](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/module-configuration-file)

### ArkTS 语法
- [ArkTS 语法适配规则](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-syntax-adaptation-rules-V5)
- [ArkTS 严格类型检查](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-strict-type-checking-V5)
- [arkts-no-obj-literals-as-types](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-no-obj-literals-as-types-V5)
- [arkts-no-untyped-obj-literals](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-no-untyped-obj-literals-V5)
- [arkts-no-noninferrable-arr-literals](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-no-noninferrable-arr-literals-V5)

### UI 开发
- [ArkUI 声明式开发](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-declarative-development-paradigm)
- [UIAbility 生命周期](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/uiability-lifecycle)
- [窗口开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/window-development-guide)

### 权限
- [权限申请指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/permissions-for-all)
- [权限列表](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/permissions-list)

## 常用命令

```bash
# 构建 HAP
npm run build

# 或直接使用 hvigor
npx hvigor assembleHap

# 清理构建缓存
npx hvigor clean

# 查看设备日志
hdc hilog | grep TAG

# 安装 HAP 到设备
hdc app install entry/build/default/outputs/default/entry-default-signed.hap
```

## 模拟器与真机调试

### 模拟器
1. 打开 DevEco Studio
2. Tools → Device Manager
3. 选择 Phone 模拟器，点击 New Emulator
4. 选择系统镜像，下载并启动

### 真机调试
1. 开启开发者模式：设置 → 关于手机 → 连续点击版本号
2. 开启 USB 调试：设置 → 系统和更新 → 开发人员选项 → USB 调试
3. 连接 USB，允许调试
4. 在 DevEco Studio 中选择设备，点击 Run

## 常见问题

### 1. 构建失败：app.json5 not found
**解决**：创建 `AppScope/app.json5` 文件。

### 2. 构建失败：Schema validate failed
**解决**：检查 `module.json5` 中必填字段是否完整（如 `startWindowIcon`）。

### 3. 应用闪退：setWindowBackgroundColor failed
**解决**：移除 `EntryAbility` 中的 `setWindowBackgroundColor` 调用，在页面组件中设置背景色。

### 4. 编译错误：Object literals cannot be used as type declarations
**解决**：将内联对象类型提取为独立的 `interface`。

### 5. 编译错误：Array literals must contain elements of only inferrable types
**解决**：为数组变量添加显式类型注解，或预定义数组元素变量。
