# HarmonyOS 项目结构、配置文件与资源管理

## 项目结构

### ArkTS 项目结构 (ArkUI) - Stage 模型 (API 9+)

```
InstantTrip/                      # 项目根目录
├── AppScope/                     # 应用级配置 (Stage模型新增)
│   ├── app.json5                 # 应用全局配置
│   └── resources/
│       └── base/
│           ├── element/
│           │   └── string.json   # 应用级字符串
│           └── media/
│               └── icon.png      # 应用图标
├── entry/                        # Entry模块
│   ├── build-profile.json5       # 模块构建配置
│   ├── hvigorfile.ts             # 模块构建脚本
│   ├── oh-package.json5          # 模块依赖管理
│   └── src/
│       └── main/
│           ├── ets/              # ArkTS源码
│           │   ├── entryability/
│           │   │   └── EntryAbility.ets
│           │   ├── pages/
│           │   │   └── Index.ets
│           │   ├── components/   # 自定义组件
│           │   ├── constants/    # 常量定义
│           │   ├── models/       # 数据模型
│           │   └── utils/        # 工具类
│           └── resources/
│               └── base/
│                   ├── element/
│                   │   ├── string.json
│                   │   └── color.json
│                   ├── media/    # 图片资源
│                   └── profile/
│                       └── main_pages.json  # 页面路由
├── build-profile.json5           # 项目级构建配置
├── hvigorfile.ts                 # 项目级构建脚本
└── oh-package.json5              # 项目级依赖管理
```

> ⚠️ **重要区别**：API 9+ 使用 **Stage 模型**，配置文件从 `config.json` 改为 `module.json5`，并新增 `AppScope` 目录存放应用级配置。

### JS UI 项目结构 (旧版 HarmonyOS)

```
src/
└── main/
    └── js/
        └── default/
            ├── pages/          # 页面目录
            │   ├── index/
            │   │   ├── index.hml  # 布局文件
            │   │   ├── index.css  # 样式文件
            │   │   └── index.js   # 逻辑文件
            │   └── detail/
            └── common/        # 公共资源
```

## 配置文件详解

### 1. AppScope/app.json5 (应用级配置)

```json5
{
  "app": {
    "bundleName": "com.instanttrip.app",    // 包名，唯一标识
    "vendor": "instanttrip",                 // 开发商
    "versionCode": 1000000,                  // 版本号（数字）
    "versionName": "1.0.0",                  // 版本名称
    "icon": "$media:icon",                   // 应用图标
    "label": "$string:app_name",             // 应用名称
    "description": "$string:app_desc",       // 应用描述
    "minAPIVersion": 12,                     // 最低API版本
    "targetAPIVersion": 12,                  // 目标API版本
    "apiReleaseType": "Release1",            // Release1/Beta1/Canary1
    "debug": false
  }
}
```

> ⚠️ **注意**：`apiReleaseType` 必须是 `"Release1"`、`"Beta1"` 或 `"Canary1"` 格式，不能只是 `"Release"`。

### 2. entry/src/main/module.json5 (模块级配置)

```json5
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": ["phone", "tablet"],
    "deliveryWithInstall": true,
    "installationFree": false,
    "pages": "$profile:main_pages",
    "abilities": [
      {
        "name": "EntryAbility",
        "srcEntry": "./ets/entryability/EntryAbility.ets",
        "description": "$string:EntryAbility_desc",
        "label": "$string:EntryAbility_label",
        "startWindowIcon": "$media:icon",       // 启动窗口图标（必需）
        "startWindowBackground": "$color:start_window_background",
        "icon": "$media:icon",                  // Ability图标
        "exported": true,
        "skills": [
          {
            "entities": ["entity.system.home"],
            "actions": ["action.system.home"]
          }
        ]
      }
    ],
    "requestPermissions": [
      {
        "name": "ohos.permission.INTERNET",
        "reason": "$string:permission_internet_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

> ⚠️ **关键字段说明**：
> - `startWindowIcon`：**必需**，缺少会导致构建失败
> - `icon`：Ability 图标，用于应用管理器等场景
> - `requestPermissions`：`user_grant` 权限必须包含 `reason` 和 `usedScene`

### 3. build-profile.json5 (构建配置)

```json5
{
  "app": {
    "signingConfigs": [],
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compileSdkVersion": "5.0.0(12)",      // 字符串格式
        "compatibleSdkVersion": "5.0.0(12)",   // 字符串格式
        "targetSdkVersion": "5.0.0(12)",       // 可选
        "runtimeOS": "HarmonyOS",
        "bundleName": "com.instanttrip.app"
      }
    ]
  }
}
```

> ⚠️ **版本号格式**：HarmonyOS 使用 `"x.x.x(API版本)"` 的字符串格式，例如 `"5.0.0(12)"`。
> 
> ⚠️ **注意**：`compileSdkVersion` 和 `compatibleSdkVersion` 不能同时在 `app` 和 `product` 中定义，只能定义在一个地方。

### 4. entry/build-profile.json5 (模块构建配置)

```json5
{
  "apiType": "stageMode",
  "buildOption": {
    "arkOptions": {
      "types": []
    }
  },
  "targets": [
    {
      "name": "default"
    }
  ]
}
```

## 资源文件

### 资源目录结构

```
resources/
├── base/                      # 默认目录（必须存在）
│   ├── element/              # 元素资源
│   │   ├── string.json       # 字符串
│   │   ├── color.json        # 颜色
│   │   ├── boolean.json      # 布尔值
│   │   ├── float.json        # 浮点数
│   │   ├── integer.json      # 整数
│   │   └── plural.json       # 复数形式
│   ├── media/                # 媒体资源
│   │   ├── icon.png          # 图标（必需）
│   │   └── background.jpg
│   ├── animation/            # 动画资源
│   └── profile/              # 配置文件
│       └── main_pages.json   # 页面路由声明
├── zh_CN/                    # 限定词目录（中文-中国）
│   ├── element/
│   └── media/
├── rawfile/                  # 原始文件（不编译）
│   └── config.json
└── en_GB/                    # 英文-英国
```

### 限定词目录

| 限定词类型 | 说明 | 示例 |
|-----------|------|------|
| 语言 | 2-3个小写字母 | zh, en, fr |
| 文字 | 首字母+3个小写字母 | Hans, Hant |
| 国家/地区 | 2-3个大写字母 | CN, US, GB |
| 横竖屏 | vertical, horizontal | vertical |
| 设备类型 | phone, tablet, car, tv | phone |
| 颜色模式 | dark, light | dark |
| 屏幕密度 | sdpi, mdpi, ldpi, xldpi... | xhdpi |

### 使用资源

```typescript
// 在 ArkTS 中使用
Text($r('app.string.title'))
  .fontColor($r('app.color.primary'))

// 在 JS UI 中使用
text('value', $r('app.string.greet'))
```

## 常见构建错误与解决方案

### 1. app.json5 file not found

**错误信息**：`Error Message: app.json5 file not found. At file: .../AppScope/app.json5`

**原因**：缺少 `AppScope/app.json5` 文件。

**解决**：创建 `AppScope/app.json5` 文件，包含应用级配置。

### 2. startWindowIcon 缺失

**错误信息**：`must have required property 'startWindowIcon'`

**原因**：`module.json5` 中 `abilities` 缺少 `startWindowIcon` 字段。

**解决**：在 Ability 配置中添加 `"startWindowIcon": "$media:icon"`。

### 3. 权限 reason 和 usedScene 缺失

**错误信息**：`The reason and usedScene attributes are mandatory for user_grant permissions.`

**原因**：`user_grant` 类型的权限必须包含 `reason` 和 `usedScene`。

**解决**：
```json5
{
  "name": "ohos.permission.LOCATION",
  "reason": "$string:permission_location_reason",
  "usedScene": {
    "abilities": ["EntryAbility"],
    "when": "inuse"
  }
}
```

### 4. SDK 版本号格式错误

**错误信息**：`compatibleSdkVersion和targetSdkVersion的值必须是字符串`

**原因**：SDK 版本号使用了数字格式。

**解决**：使用字符串格式 `"5.0.0(12)"`。

### 5. apiReleaseType 格式错误

**错误信息**：`must match pattern "^(Canary[1-9]\d*)|(Beta[1-9]\d*)|(Release[1-9]\d*)$"`

**原因**：`apiReleaseType` 值格式不正确。

**解决**：使用 `"Release1"`、`"Beta1"` 或 `"Canary1"`。

### 6. 资源引用未定义

**错误信息**：`The resource reference '$media:icon' is not defined`

**原因**：引用的资源文件不存在。

**解决**：在 `resources/base/media/` 目录下放置对应的资源文件。

## Stage 模型 vs FA 模型

| 特性 | Stage 模型 (API 9+) | FA 模型 (旧版) |
|------|---------------------|----------------|
| 配置文件 | `module.json5` | `config.json` |
| 应用级配置 | `AppScope/app.json5` | 无 |
| Ability 类型 | UIAbility / ExtensionAbility | PageAbility / ServiceAbility |
| 窗口管理 | WindowStage | 无 |
| 生命周期 | 更精细 | 简单 |
| 推荐度 | ✅ 推荐 | ⚠️ 已废弃 |

## 参考

- [华为官方文档 - 应用配置文件](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-hvigor-build-profile-app)
- [Stage 模型开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/stage-model-development-overview)
