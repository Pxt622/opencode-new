# HarmonyOS 权限管理

## 权限声明

在 `entry/src/main/module.json5` 中声明所需权限：

```json5
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.INTERNET",
        "reason": "$string:permission_internet_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.LOCATION",
        "reason": "$string:permission_location_reason",
        "usedScene": {
          "abilities": ["EntryAbility"],
          "when": "inuse"
        }
      }
    ]
  }
}
```

> ⚠️ **重要**：`user_grant` 类型的权限必须包含 `reason` 和 `usedScene` 字段，否则构建会失败。

## 权限类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `system_grant` | 系统授权，安装时自动授予 | `ohos.permission.INTERNET` |
| `user_grant` | 用户授权，需要运行时申请 | `ohos.permission.LOCATION` |

## 常用权限

| 权限名称 | 说明 | 类型 |
|---------|------|------|
| `ohos.permission.INTERNET` | 允许应用访问网络 | system_grant |
| `ohos.permission.GET_NETWORK_INFO` | 获取网络信息 | system_grant |
| `ohos.permission.CAMERA` | 允许应用使用相机 | user_grant |
| `ohos.permission.RECORD_AUDIO` | 允许应用录音 | user_grant |
| `ohos.permission.READ_CONTACTS` | 读取联系人 | user_grant |
| `ohos.permission.WRITE_CONTACTS` | 写入联系人 | user_grant |
| `ohos.permission.ACCESS_LOCATION` | 获取位置信息 | user_grant |
| `ohos.permission.LOCATION` | 获取位置信息（API 12+） | user_grant |
| `ohos.permission.DISTRIBUTED_DATASYNC` | 分布式数据同步 | user_grant |

## 字符串资源定义

在 `resources/base/element/string.json` 中定义权限说明：

```json
{
  "string": [
    {
      "name": "permission_internet_reason",
      "value": "用于获取旅行路线、地图数据和AI推荐服务"
    },
    {
      "name": "permission_location_reason",
      "value": "用于定位当前位置和规划周边游路线"
    }
  ]
}
```

## usedScene 字段说明

```json5
{
  "usedScene": {
    "abilities": ["EntryAbility"],  // 使用该权限的 Ability 列表
    "when": "inuse"                 // 使用时机：inuse(使用时) / always(始终)
  }
}
```

| 字段 | 说明 | 可选值 |
|------|------|--------|
| `abilities` | 使用该权限的 Ability 名称数组 | `["EntryAbility"]` |
| `when` | 权限使用时机 | `"inuse"` / `"always"` |

## 动态权限申请

```typescript
import { abilityAccessCtrl, PermissionRequestResult } from '@kit.AbilityKit'
import { bundleManager } from '@kit.BundleKit'

// 获取权限管理类
let atManager = abilityAccessCtrl.createAtManager()

// 检查权限状态
async function checkPermission(): Promise<boolean> {
  let result = await atManager.checkAccessToken(
    bundleManager.getBundleNameByUid(getContext(this).applicationInfo.uid),
    'ohos.permission.CAMERA'
  )
  return result === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED
}

// 请求权限
async function requestPermission(): Promise<boolean> {
  let requestResult: PermissionRequestResult = await atManager.requestPermissionsFromUser(
    getContext(this),
    ['ohos.permission.CAMERA', 'ohos.permission.LOCATION']
  )
  return requestResult.authResults.every((result) => result === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED)
}
```

## 权限使用原则

1. **权限申请最小化**：不要申请与功能无关的权限
2. **权限申请完整**：所有需要的权限都要声明
3. **满足用户可知**：敏感权限的目的需要真实准确告知用户
4. **权限就近申请**：在用户触发相关功能时就近提示授权
5. **权限不扩散**：未授权的权限不能提供给其他应用使用

## 常见构建错误

### reason 和 usedScene 缺失

**错误信息**：`The reason and usedScene attributes are mandatory for user_grant permissions.`

**原因**：`user_grant` 权限缺少 `reason` 或 `usedScene` 字段。

**解决**：为所有 `user_grant` 权限添加完整的 `reason` 和 `usedScene` 配置。

## 参考

- [华为官方文档 - 权限申请指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/permissions-for-all)
- [权限列表](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/permissions-list)
