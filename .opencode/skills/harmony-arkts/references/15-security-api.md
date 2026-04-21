# HarmonyOS 安全 API 开发指南

## 安全架构概述

HarmonyOS 提供了一套完整的安全框架，保护用户数据隐私和系统安全。主要的安全组件包括：

| 组件 | 功能描述 | 主要 API |
|------|---------|---------|
| **HUKS** (HarmonyOS Universal Keystore System) | 统一的密钥管理系统，提供密钥生成、存储、使用和销毁 | `@ohos.security.huks` |
| **UserAuth** | 用户身份认证框架，支持生物特征（指纹、人脸）和密码认证 | `@ohos.userIAM.userAuth` |
| **CryptoFramework** | 加密算法框架，提供对称/非对称加密、哈希、数字签名等 | `@ohos.security.cryptoFramework` |
| **AccessToken** | 访问令牌管理，控制应用权限和资源访问 | `@ohos.security.accessToken` |
| **AppAccount** | 应用账号管理，支持多账号和 OAuth 认证 | `@ohos.account.appAccount` |

## HUKS (HarmonyOS Universal Keystore System)

### HUKS 概述

HUKS 是 HarmonyOS 的统一密钥管理系统，提供安全的密钥存储和密码学操作。所有密钥操作都在安全环境中执行，密钥内容不会暴露给应用。

### 密钥类型

| 密钥类型 | 用途 | 特点 |
|---------|------|------|
| **对称密钥** | AES、SM4 加密 | 加解密使用相同密钥，速度快 |
| **非对称密钥** | RSA、ECC、SM2 | 公私钥对，支持加密和签名 |
| **HMAC 密钥** | 消息认证 | 用于生成消息认证码 |
| **推导密钥** | 密钥派生 | 从主密钥派生子密钥 |

### 基本使用流程

```typescript
import huks from '@ohos.security.huks'

// 1. 生成密钥
async function generateKey(): Promise<void> {
  let keyAlias = 'my_aes_key'
  let properties: huks.HuksOptions = {
    properties: [
      {
        tag: huks.HuksTag.HUKS_TAG_ALGORITHM,
        value: huks.HuksKeyAlg.HUKS_ALG_AES
      },
      {
        tag: huks.HuksTag.HUKS_TAG_KEY_SIZE,
        value: huks.HuksKeySize.HUKS_AES_KEY_SIZE_256
      },
      {
        tag: huks.HuksTag.HUKS_TAG_PURPOSE,
        value: huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_ENCRYPT | 
               huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_DECRYPT
      },
      {
        tag: huks.HuksTag.HUKS_TAG_PADDING,
        value: huks.HuksKeyPadding.HUKS_PADDING_PKCS7
      },
      {
        tag: huks.HuksTag.HUKS_TAG_BLOCK_MODE,
        value: huks.HuksCipherMode.HUKS_MODE_CBC
      },
      {
        tag: huks.HuksTag.HUKS_TAG_DIGEST,
        value: huks.HuksKeyDigest.HUKS_DIGEST_SHA256
      }
    ]
  }

  try {
    await huks.generateKeyItem(keyAlias, properties)
    console.log('密钥生成成功')
  } catch (error) {
    console.error(`密钥生成失败: ${error.message}`)
  }
}

// 2. 使用密钥加密数据
async function encryptData(plainText: string): Promise<Uint8Array> {
  let keyAlias = 'my_aes_key'
  let cipherText: Uint8Array
  
  // 加密参数
  let encryptOptions: huks.HuksOptions = {
    properties: [
      {
        tag: huks.HuksTag.HUKS_TAG_ALGORITHM,
        value: huks.HuksKeyAlg.HUKS_ALG_AES
      },
      {
        tag: huks.HuksTag.HUKS_TAG_PURPOSE,
        value: huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_ENCRYPT
      },
      {
        tag: huks.HuksTag.HUKS_TAG_PADDING,
        value: huks.HuksKeyPadding.HUKS_PADDING_PKCS7
      },
      {
        tag: huks.HuksTag.HUKS_TAG_BLOCK_MODE,
        value: huks.HuksCipherMode.HUKS_MODE_CBC
      },
      {
        tag: huks.HuksTag.HUKS_TAG_IV,
        value: new Uint8Array(16) // 初始化向量
      }
    ],
    inData: new TextEncoder().encode(plainText)
  }

  try {
    const result = await huks.initSession(keyAlias, encryptOptions)
    const handle = result.handle
    
    // 分段加密（如果需要）
    cipherText = await huks.finishSession(handle, encryptOptions)
    
    await huks.abortSession(handle, encryptOptions)
    return cipherText
  } catch (error) {
    console.error(`加密失败: ${error.message}`)
    throw error
  }
}

// 3. 使用密钥解密数据
async function decryptData(cipherText: Uint8Array): Promise<string> {
  let keyAlias = 'my_aes_key'
  let plainText: Uint8Array
  
  // 解密参数
  let decryptOptions: huks.HuksOptions = {
    properties: [
      {
        tag: huks.HuksTag.HUKS_TAG_ALGORITHM,
        value: huks.HuksKeyAlg.HUKS_ALG_AES
      },
      {
        tag: huks.HuksTag.HUKS_TAG_PURPOSE,
        value: huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_DECRYPT
      },
      {
        tag: huks.HuksTag.HUKS_TAG_PADDING,
        value: huks.HuksKeyPadding.HUKS_PADDING_PKCS7
      },
      {
        tag: huks.HuksTag.HUKS_TAG_BLOCK_MODE,
        value: huks.HuksCipherMode.HUKS_MODE_CBC
      },
      {
        tag: huks.HuksTag.HUKS_TAG_IV,
        value: new Uint8Array(16) // 必须与加密时相同
      }
    ],
    inData: cipherText
  }

  try {
    const result = await huks.initSession(keyAlias, decryptOptions)
    const handle = result.handle
    
    plainText = await huks.finishSession(handle, decryptOptions)
    
    await huks.abortSession(handle, decryptOptions)
    return new TextDecoder().decode(plainText)
  } catch (error) {
    console.error(`解密失败: ${error.message}`)
    throw error
  }
}

// 4. 删除密钥
async function deleteKey(): Promise<void> {
  let keyAlias = 'my_aes_key'
  
  try {
    await huks.deleteKeyItem(keyAlias, null)
    console.log('密钥删除成功')
  } catch (error) {
    console.error(`密钥删除失败: ${error.message}`)
  }
}
```

### 密钥导入与导出

```typescript
// 导入外部密钥
async function importKey(): Promise<void> {
  let keyAlias = 'imported_key'
  
  // 准备要导入的密钥材料（示例）
  let keyMaterial = new Uint8Array(32) // 256位 AES 密钥
  
  let importOptions: huks.HuksOptions = {
    properties: [
      {
        tag: huks.HuksTag.HUKS_TAG_ALGORITHM,
        value: huks.HuksKeyAlg.HUKS_ALG_AES
      },
      {
        tag: huks.HuksTag.HUKS_TAG_KEY_SIZE,
        value: huks.HuksKeySize.HUKS_AES_KEY_SIZE_256
      },
      {
        tag: huks.HuksTag.HUKS_TAG_PURPOSE,
        value: huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_ENCRYPT
      }
    ],
    inData: keyMaterial
  }
  
  try {
    await huks.importKeyItem(keyAlias, importOptions)
    console.log('密钥导入成功')
  } catch (error) {
    console.error(`密钥导入失败: ${error.message}`)
  }
}

// 导出公钥（仅适用于非对称密钥）
async function exportPublicKey(): Promise<Uint8Array> {
  let keyAlias = 'my_rsa_key'
  
  try {
    const result = await huks.exportKeyItem(keyAlias, null)
    return result.outData
  } catch (error) {
    console.error(`导出公钥失败: ${error.message}`)
    throw error
  }
}
```

### 密钥访问控制

```typescript
// 设置密钥访问策略
async function setKeyAccessPolicy(): Promise<void> {
  let keyAlias = 'sensitive_key'
  
  let keyProperties: huks.HuksOptions = {
    properties: [
      {
        tag: huks.HuksTag.HUKS_TAG_ALGORITHM,
        value: huks.HuksKeyAlg.HUKS_ALG_RSA
      },
      {
        tag: huks.HuksTag.HUKS_TAG_KEY_SIZE,
        value: huks.HuksKeySize.HUKS_RSA_KEY_SIZE_2048
      },
      {
        tag: huks.HuksTag.HUKS_TAG_PURPOSE,
        value: huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_SIGN |
               huks.HuksKeyPurpose.HUKS_KEY_PURPOSE_VERIFY
      },
      // 访问控制策略
      {
        tag: huks.HuksTag.HUKS_TAG_ACCESSIBILITY,
        value: huks.HuksKeyAccessibility.HUKS_ACCESSIBLE_AFTER_FIRST_UNLOCK
      },
      {
        tag: huks.HuksTag.HUKS_TAG_AUTH_PURPOSE,
        value: huks.HuksAuthPurpose.HUKS_AUTH_PURPOSE_ENCRYPT
      },
      {
        tag: huks.HuksTag.HUKS_TAG_ALLOWED_DATE,
        value: new Date('2026-12-31').getTime() // 过期时间
      }
    ]
  }
  
  try {
    await huks.generateKeyItem(keyAlias, keyProperties)
    console.log('带访问控制的密钥生成成功')
  } catch (error) {
    console.error(`密钥生成失败: ${error.message}`)
  }
}
```

## UserAuth (用户认证)

### UserAuth 概述

UserAuth 提供统一的用户身份认证框架，支持多种认证方式：
- 密码认证
- 生物特征认证（指纹、人脸）
- 多因素认证

### 认证等级

| 认证等级 | 描述 | 适用场景 |
|---------|------|---------|
| **S1** | 低安全等级 | 应用解锁、非敏感操作 |
| **S2** | 中安全等级 | 支付确认、中等敏感操作 |
| **S3** | 高安全等级 | 金融交易、高敏感操作 |
| **S4** | 最高安全等级 | 系统关键操作、设备管理 |

### 生物特征认证

```typescript
import userIAM_userAuth from '@ohos.userIAM.userAuth'

// 检查设备支持的认证类型
async function checkAuthSupport(): Promise<void> {
  try {
    const version = userIAM_userAuth.getVersion()
    console.log(`UserAuth 版本: ${version}`)
    
    // 获取支持的认证类型
    const authTypes = await userIAM_userAuth.getAvailableStatus(
      userIAM_userAuth.UserAuthType.FACE, // 人脸认证
      userIAM_userAuth.AuthTrustLevel.ATL1 // 信任等级
    )
    
    console.log(`支持的认证类型: ${JSON.stringify(authTypes)}`)
  } catch (error) {
    console.error(`检查认证支持失败: ${error.message}`)
  }
}

// 执行认证
async function performAuthentication(): Promise<boolean> {
  const auth = new userIAM_userAuth.UserAuth()
  
  // 认证参数
  const authParam: userIAM_userAuth.AuthParam = {
    challenge: new Uint8Array(32), // 随机挑战值
    authType: userIAM_userAuth.UserAuthType.FACE, // 认证类型
    authTrustLevel: userIAM_userAuth.AuthTrustLevel.ATL3 // 信任等级
  }
  
  // 认证回调
  const authCallback: userIAM_userAuth.AuthEvent = {
    onResult: (result: number, extraInfo: userIAM_userAuth.AuthResultInfo) => {
      console.log(`认证结果: ${result}`)
      console.log(`剩余尝试次数: ${extraInfo.remainTimes}`)
      console.log(`锁定时间: ${extraInfo.freezingTime}`)
      
      if (result === userIAM_userAuth.ResultCode.SUCCESS) {
        console.log('认证成功')
        // 获取认证令牌
        const token = extraInfo.token
        console.log(`认证令牌: ${Array.from(token).map(b => b.toString(16).padStart(2, '0')).join('')}`)
      } else {
        console.error(`认证失败，错误码: ${result}`)
      }
    }
  }
  
  try {
    // 开始认证
    const authTask = auth.auth(authParam, authCallback)
    
    // 可以取消认证（如果需要）
    // authTask.cancel()
    
    return true
  } catch (error) {
    console.error(`认证过程失败: ${error.message}`)
    return false
  }
}

// 检查认证状态
async function checkAuthStatus(): Promise<void> {
  try {
    const status = await userIAM_userAuth.getAuthStatus(
      userIAM_userAuth.UserAuthType.FINGERPRINT
    )
    
    console.log(`指纹认证状态: ${status}`)
    
    if (status === userIAM_userAuth.AuthStatus.AUTH_STATUS_ENROLLED) {
      console.log('已注册指纹')
    } else if (status === userIAM_userAuth.AuthStatus.AUTH_STATUS_NOT_ENROLLED) {
      console.log('未注册指纹')
    } else if (status === userIAM_userAuth.AuthStatus.AUTH_STATUS_LOCKED) {
      console.log('认证已锁定')
    }
  } catch (error) {
    console.error(`检查认证状态失败: ${error.message}`)
  }
}
```

### 多因素认证

```typescript
// 两步验证示例
class TwoFactorAuth {
  private auth = new userIAM_userAuth.UserAuth()
  
  async authenticateWith2FA(): Promise<boolean> {
    // 第一步：密码认证
    const passwordResult = await this.authenticateWithPassword()
    
    if (!passwordResult) {
      return false
    }
    
    // 第二步：生物特征认证
    const biometricResult = await this.authenticateWithBiometric()
    
    return passwordResult && biometricResult
  }
  
  private async authenticateWithPassword(): Promise<boolean> {
    // 这里应该连接到应用自己的密码验证系统
    // 示例：模拟密码验证
    return new Promise((resolve) => {
      setTimeout(() => {
        // 假设密码验证成功
        resolve(true)
      }, 1000)
    })
  }
  
  private async authenticateWithBiometric(): Promise<boolean> {
    const authParam: userIAM_userAuth.AuthParam = {
      challenge: new Uint8Array(32),
      authType: userIAM_userAuth.UserAuthType.FACE,
      authTrustLevel: userIAM_userAuth.AuthTrustLevel.ATL3
    }
    
    return new Promise((resolve) => {
      const authCallback: userIAM_userAuth.AuthEvent = {
        onResult: (result: number) => {
          resolve(result === userIAM_userAuth.ResultCode.SUCCESS)
        }
      }
      
      this.auth.auth(authParam, authCallback)
    })
  }
}
```

## CryptoFramework (加密框架)

### CryptoFramework 概述

CryptoFramework 提供通用的密码学算法实现，包括：
- 对称加密（AES、SM4）
- 非对称加密（RSA、ECC、SM2）
- 哈希算法（SHA-256、SM3）
- 消息认证码（HMAC）
- 数字签名
- 密钥协商

### 对称加密

```typescript
import cryptoFramework from '@ohos.security.cryptoFramework'

// AES-CBC 加密
async function aesEncrypt(plainText: string, key: Uint8Array): Promise<Uint8Array> {
  // 创建对称密钥生成器
  let symKeyGenerator = cryptoFramework.createSymKeyGenerator('AES256')
  
  // 转换密钥
  let keyBlob: cryptoFramework.DataBlob = { data: key }
  let symKey = await symKeyGenerator.convertKey(keyBlob)
  
  // 创建加密器
  let cipher = cryptoFramework.createCipher('AES256|CBC|PKCS7')
  
  // 设置加密参数
  let iv = new Uint8Array(16) // 初始化向量
  let paramsSpec = cryptoFramework.IvParamsSpec({ iv: { data: iv } })
  
  // 初始化加密器
  await cipher.init(cryptoFramework.CryptoMode.ENCRYPT_MODE, symKey, paramsSpec)
  
  // 加密数据
  let input: cryptoFramework.DataBlob = { data: new TextEncoder().encode(plainText) }
  let output = await cipher.doFinal(input)
  
  return output.data
}

// AES-CBC 解密
async function aesDecrypt(cipherText: Uint8Array, key: Uint8Array): Promise<string> {
  let symKeyGenerator = cryptoFramework.createSymKeyGenerator('AES256')
  let keyBlob: cryptoFramework.DataBlob = { data: key }
  let symKey = await symKeyGenerator.convertKey(keyBlob)
  
  let cipher = cryptoFramework.createCipher('AES256|CBC|PKCS7')
  
  // 使用相同的 IV
  let iv = new Uint8Array(16)
  let paramsSpec = cryptoFramework.IvParamsSpec({ iv: { data: iv } })
  
  await cipher.init(cryptoFramework.CryptoMode.DECRYPT_MODE, symKey, paramsSpec)
  
  let input: cryptoFramework.DataBlob = { data: cipherText }
  let output = await cipher.doFinal(input)
  
  return new TextDecoder().decode(output.data)
}
```

### 非对称加密与数字签名

```typescript
// RSA 密钥生成与签名
async function rsaSignAndVerify(): Promise<void> {
  // 1. 生成 RSA 密钥对
  let rsaKeyGenerator = cryptoFramework.createAsyKeyGenerator('RSA2048|PRIMES_2')
  let keyPair = await rsaKeyGenerator.generateKeyPair()
  
  console.log('RSA 密钥对生成成功')
  
  // 2. 创建签名器
  let signer = cryptoFramework.createSign('RSA2048|PSS|SHA256')
  
  // 3. 初始化签名器
  let signOptions: cryptoFramework.SignSpec = {
    data: { data: new Uint8Array([1, 2, 3, 4, 5]) }
  }
  
  await signer.init(cryptoFramework.SignMode.SIGN_MODE, keyPair.priKey, signOptions)
  
  // 4. 生成签名
  let signature = await signer.sign(signOptions)
  console.log(`签名: ${Array.from(signature.data).map(b => b.toString(16).padStart(2, '0')).join('')}`)
  
  // 5. 创建验签器
  let verifier = cryptoFramework.createVerify('RSA2048|PSS|SHA256')
  
  // 6. 初始化验签器
  await verifier.init(cryptoFramework.SignMode.VERIFY_MODE, keyPair.pubKey, signOptions)
  
  // 7. 验证签名
  let verifyOptions: cryptoFramework.SignSpec = {
    data: { data: new Uint8Array([1, 2, 3, 4, 5]) },
    signature: signature
  }
  
  let isValid = await verifier.verify(verifyOptions)
  console.log(`签名验证结果: ${isValid ? '有效' : '无效'}`)
}
```

### 哈希算法

```typescript
// 计算 SHA-256 哈希值
async function computeHash(data: string): Promise<Uint8Array> {
  // 创建哈希器
  let hash = cryptoFramework.createHash('SHA256')
  
  // 初始化
  await hash.init()
  
  // 更新数据（支持分段）
  let input: cryptoFramework.DataBlob = { data: new TextEncoder().encode(data) }
  await hash.update(input)
  
  // 计算最终哈希值
  let output = await hash.doFinal(input)
  
  console.log(`SHA-256 哈希值: ${Array.from(output.data).map(b => b.toString(16).padStart(2, '0')).join('')}`)
  return output.data
}

// 计算文件哈希（分段处理）
async function computeFileHash(fileData: Uint8Array): Promise<Uint8Array> {
  let hash = cryptoFramework.createHash('SHA256')
  await hash.init()
  
  // 分段处理大数据
  const chunkSize = 4096
  for (let i = 0; i < fileData.length; i += chunkSize) {
    const chunk = fileData.slice(i, i + chunkSize)
    await hash.update({ data: chunk })
  }
  
  let output = await hash.doFinal({ data: new Uint8Array(0) })
  return output.data
}
```

### 密钥协商

```typescript
// ECDH 密钥协商
async function performKeyAgreement(): Promise<Uint8Array> {
  // 双方各自生成 ECC 密钥对
  let eccKeyGenerator = cryptoFramework.createAsyKeyGenerator('ECC256')
  let keyPairA = await eccKeyGenerator.generateKeyPair()
  let keyPairB = await eccKeyGenerator.generateKeyPair()
  
  // 甲方使用自己的私钥和乙方的公钥生成共享密钥
  let keyAgreement = cryptoFramework.createKeyAgreement('ECC256')
  await keyAgreement.init(keyPairA.priKey)
  
  let sharedKeyA = await keyAgreement.generateSecret(keyPairB.pubKey)
  
  // 乙方使用自己的私钥和甲方的公钥生成共享密钥
  await keyAgreement.init(keyPairB.priKey)
  let sharedKeyB = await keyAgreement.generateSecret(keyPairA.pubKey)
  
  // 双方生成的共享密钥应该相同
  console.log(`密钥协商成功: ${sharedKeyA.data.length === sharedKeyB.data.length ? '是' : '否'}`)
  
  return sharedKeyA.data
}
```

## 安全最佳实践

### 1. 密钥管理

- **使用 HUKS 存储所有密钥**：不要将密钥存储在文件或 SharedPreferences 中
- **定期轮换密钥**：为长期使用的数据设置密钥过期时间
- **分级管理密钥**：根据敏感性使用不同的密钥访问策略
- **安全销毁密钥**：不再使用的密钥及时从 HUKS 中删除

### 2. 认证设计

- **渐进式认证**：根据操作风险级别要求不同的认证强度
- **多因素认证**：对敏感操作要求多种认证方式
- **认证上下文**：记录认证事件用于审计和安全分析
- **失败处理**：限制连续失败次数，防止暴力破解

### 3. 数据加密

- **传输层加密**：所有网络通信使用 TLS
- **存储加密**：敏感数据在存储前加密
- **内存安全**：及时清理内存中的敏感数据
- **算法选择**：优先使用国密算法（SM2/SM3/SM4）或行业标准算法

### 4. 权限控制

```typescript
import accessToken from '@ohos.security.accessToken'

// 检查应用权限
async function checkPermissions(): Promise<void> {
  try {
    // 获取当前应用的 AccessToken ID
    const tokenId = accessToken.getLocalTokenId()
    
    // 检查特定权限
    const hasPermission = await accessToken.verifyAccessToken(
      tokenId,
      'ohos.permission.CAMERA'
    )
    
    console.log(`相机权限: ${hasPermission === accessToken.PermissionState.PERMISSION_GRANTED ? '已授予' : '未授予'}`)
    
    // 请求权限（如果未授予）
    if (hasPermission !== accessToken.PermissionState.PERMISSION_GRANTED) {
      // 这里应该触发系统的权限请求对话框
      console.log('需要请求相机权限')
    }
  } catch (error) {
    console.error(`检查权限失败: ${error.message}`)
  }
}

// 动态申请权限
async function requestPermissions(): Promise<void> {
  const permissions = [
    'ohos.permission.CAMERA',
    'ohos.permission.MICROPHONE',
    'ohos.permission.READ_MEDIA'
  ]
  
  // 在实际应用中，这里应该调用系统的权限请求 API
  // 示例代码仅展示概念
  console.log(`需要申请的权限: ${permissions.join(', ')}`)
}
```

### 5. 安全审计

```typescript
// 记录安全事件
class SecurityLogger {
  static logSecurityEvent(eventType: string, details: any): void {
    console.log(`[安全事件] ${new Date().toISOString()} - ${eventType}: ${JSON.stringify(details)}`)
    
    // 在实际应用中，这里应该：
    // 1. 将日志写入安全存储
    // 2. 上传到安全服务器（如果配置了）
    // 3. 触发相关告警（如果需要）
  }
  
  static logAuthAttempt(userId: string, success: boolean, method: string): void {
    this.logSecurityEvent('AUTH_ATTEMPT', {
      userId,
      success,
      method,
      timestamp: Date.now(),
      deviceId: this.getDeviceId()
    })
  }
  
  static logKeyOperation(keyAlias: string, operation: string, success: boolean): void {
    this.logSecurityEvent('KEY_OPERATION', {
      keyAlias: keyAlias.substring(0, 8) + '...', // 部分掩码
      operation,
      success,
      timestamp: Date.now()
    })
  }
  
  private static getDeviceId(): string {
    // 获取设备标识（示例）
    return 'device_' + Math.random().toString(36).substring(2, 10)
  }
}

// 使用示例
SecurityLogger.logAuthAttempt('user123', true, 'FACE')
SecurityLogger.logKeyOperation('payment_key', 'ENCRYPT', true)
```

## 调试与故障排除

### 常见错误码

| 错误码 | 模块 | 描述 | 解决方案 |
|--------|------|------|---------|
| 401 | 通用 | 参数错误 | 检查 API 调用参数 |
| 801 | 通用 | 不支持的功能 | 检查设备能力和系统版本 |
| 17700001 | HUKS | 密钥不存在 | 检查密钥别名是否正确 |
| 17700002 | HUKS | 密钥已存在 | 使用不同的密钥别名或先删除旧密钥 |
| 17700003 | HUKS | 密钥操作失败 | 检查密钥属性和访问权限 |
| 12500001 | UserAuth | 认证类型不支持 | 检查设备支持的生物特征类型 |
| 12500002 | UserAuth | 认证失败 | 检查用户是否已注册相应生物特征 |

### 调试工具

```typescript
// 安全模块调试工具
class SecurityDebugger {
  static async testHUKS(): Promise<void> {
    try {
      console.log('=== HUKS 功能测试 ===')
      
      // 测试密钥生成
      const keyAlias = 'test_key_' + Date.now()
      console.log(`测试密钥: ${keyAlias}`)
      
      // ... 执行 HUKS 操作测试
      
      console.log('HUKS 功能测试完成')
    } catch (error) {
      console.error(`HUKS 测试失败: ${error.message}`)
    }
  }
  
  static async testCryptoFramework(): Promise<void> {
    try {
      console.log('=== CryptoFramework 功能测试 ===')
      
      // 测试哈希功能
      const hash = await computeHash('test data')
      console.log(`哈希测试完成: ${hash.length} bytes`)
      
      // 测试加密功能
      const key = new Uint8Array(32)
      const encrypted = await aesEncrypt('test message', key)
      const decrypted = await aesDecrypt(encrypted, key)
      console.log(`加解密测试: ${decrypted === 'test message' ? '成功' : '失败'}`)
      
      console.log('CryptoFramework 功能测试完成')
    } catch (error) {
      console.error(`CryptoFramework 测试失败: ${error.message}`)
    }
  }
}
```

## 参考

- [华为官方文档 - HUKS 开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/huks-overview)
- [华为官方文档 - UserAuth 开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/userauth-overview)
- [华为官方文档 - CryptoFramework 开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/cryptoframework-overview)
- [安全开发最佳实践](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/security-best-practices)

> 注意：安全 API 的使用需要严格遵守相关法律法规和隐私政策。在处理用户敏感数据时，必须获取用户明确同意并遵循最小必要原则。