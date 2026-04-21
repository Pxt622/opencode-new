# HarmonyOS 命令行工具完整指南

## 命令行工具概览

HarmonyOS 提供了丰富的命令行工具链，用于开发、调试、测试和发布。掌握这些工具可以极大提升开发效率。

| 工具 | 主要功能 | 适用场景 |
|------|---------|---------|
| **hdc** | 设备管理、日志查看、应用管理 | 调试、日志分析、设备模拟 |
| **aa** | 自动化测试工具 | 单元测试、集成测试、性能测试 |
| **bm** | 二进制文件处理 | 签名、混淆、资源处理 |
| **app-check** | 应用合规性检查 | 发布前质量检查 |
| **binary-sign** | 应用签名 | 发布包签名、证书管理 |

## hdc (HarmonyOS Device Connector)

### 基础连接命令

```bash
# 连接设备
hdc list targets

# 查看设备信息
hdc device list

# 查看设备详细信息
hdc shell getprop

# 查看系统属性
hdc shell param get const.product.model
hdc shell param get ohos.build.version
```

### 日志管理

```bash
# 实时查看日志
hdc shell hilog

# 过滤特定日志
hdc shell hilog | grep "MyTag"

# 查看错误日志
hdc shell hilog -l E

# 清空日志
hdc shell hilog -g

# 保存日志到文件
hdc hilog > app_log.txt

# 过滤特定模块的日志
hdc shell hilog | grep -E "Network|Image|Database"
```

### 应用管理

```bash
# 安装应用
hdc install app -p /path/to/app.hap

# 卸载应用
hdc uninstall package com.example.app

# 强制停止应用
hdc shell force-stop com.example.app

# 查看应用列表
hdc shell aa dump -a

# 查看特定应用信息
hdc shell aa dump -n com.example.myapp

# 启动应用
hdc shell aa start -n com.example.myapp

# 停止应用
hdc shell aa stop com.example.myapp
```

### 文件操作

```bash
# 上传文件到设备
hdc file send /local/file /remote/path

# 从设备下载文件
hdc file recv /remote/path /local/path

# 查看设备文件
hdc shell ls -l /data/app

# 删除设备文件
hdc shell rm /data/app/temp/file.txt

# 创建设备目录
hdc shell mkdir -p /data/app/newdir

# 查看文件权限
hdc shell ls -l /data/app/

# 修改文件权限
hdc shell chmod 644 /data/app/file.txt
```

### 性能监控

```bash
# 查看CPU信息
hdc shell top -n 1 | grep com.example.app

# 查看内存信息
hdc shell dumpsys meminfo com.example.app

# 查看进程信息
hdc shell ps -ef | grep myapp

# 查看线程信息
hdc shell ps -T | grep myapp

# 查看网络连接
hdc shell netstat -an | grep ESTABLISHED

# 查看电池信息
hdc shell dumpsys battery
```

### Shell 命令

```bash
# 执行Shell命令
hdc shell "ls -l /data/app"

# 多命令执行
hdc shell "ls -l /data/app && cat README.md"

# Root权限执行
hdc shell root "rm /data/system/Temp/*"

# 环境变量
hdc shell "echo $PATH"

# 切换用户
hdc shell su user0
```

### 网络调试

```bash
# 查看网络状态
hdc shell netcfg -v

# 查看网络接口
hdc shell ifconfig -a

# 测试网络连接
hdc shell ping -c 4 developer.huawei.com

# 查看DNS解析
hdc shell getprop net.dns1

# 查看路由表
hdc shell ip route
```

### 应用调试

```bash
# 查看应用日志
hdc shell "log -t MyTag -v"

# 转储应用日志
hdc shell "logcat -b -t MyTag > app_log.txt"

# 查看应用状态
hdc shell dumpsys window | grep -E "com.example.app"

# 查看Ability信息
hdc shell aa dump -a | grep -A 5 "com.example.myapp"

# 清除应用数据
hdc shell rm -rf /data/app/com.example.app
```

### 截图与录屏

```bash
# 截图
hdc shell screencap -p /sdcard/screen.png

# 录屏
hdc shell screenrecord /sdcard/record.mp4

# 停止录屏
hdc shell pkill screenrecord

# 设置分辨率
hdc shell wm size 1080x1920
```

## aa (App Analyzer) 测试框架

### 单元测试

```typescript
import { describe, test, expect, beforeEach, afterEach } from '@ohos.hypium'
import driver from '@ohos.hypium'

describe('UserService', () => {
  let driver: any
  
  beforeEach(async () => {
    // 启动应用
    driver = await driver.createDriver('com.example.app')
    await driver.start()
  })
  
  afterEach(async () => {
    // 关闭应用
    await driver.quit()
  })
  
  test('should login successfully', async () => {
    // 导航到登录页面
    await driver.findAndClick(By.text('登录'))
    
    // 输入用户名
    await driver.findAndSendKeys(By.id('username'), 'testuser')
    
    // 输入密码
    await driver.findAndSendKeys(By.id('password'), 'testpass')
    
    // 点击登录按钮
    await driver.findAndClick(By.text('登录'))
    
    // 验证登录成功
    const element = await driver.findElement(By.text('欢迎'))
    expect(element).toExist()
  })
  
  test('should display user info', async () => {
    // 导航到用户页面
    await driver.findAndClick(By.text('我的'))
    
    // 验证用户信息显示
    const nameElement = await driver.findElement(By.id('userName'))
    const nameText = await nameElement.text()
    expect(nameText).toBe('testuser')
  })
})
```

### 集成测试

```typescript
import { suite, test, expect, beforeAll, afterAll } from '@ohos.hypium'

suite('E2E Commerce Tests', () => {
  let driver: any
  
  beforeAll(async () => {
    driver = await driver.createDriver('com.example.ecommerce')
    await driver.start()
    
    // 登录
    await login(driver)
  })
  
  afterAll(async () => {
    // 退出登录
    await logout(driver)
    await driver.quit()
  })
  
  test('complete purchase flow', async () => {
    // 浏览商品
    await driver.findAndClick(By.text('商品'))
    
    // 添加到购物车
    await driver.findAndClick(By.text('加入购物车'))
    
    // 查看购物车
    await driver.findAndClick(By.text('购物车'))
    
    // 结算
    await driver.findAndClick(By.text('结算'))
    
    // 支付付
    await driver.findAndClick(By.text('微信支付'))
    
    // 验证订单创建成功
    const element = await driver.findElement(By.text('订单提交成功'))
    expect(element).toExist()
  })
})
```

### 性能测试

```typescript
import { suite, test, expect } from '@ohos.hypium'

suite('Performance Tests', () => {
  let driver: any
  
  beforeAll(async () => {
    driver = await driver.createDriver('com.example.app')
    await driver.start()
  })
  
  afterAll(async () => {
    await driver.quit()
  })
  
  test('page load performance', async () => {
    const startTime = Date.now()
    
    // 访问首页
    await driver.getPage('https://example.com/page')
    
    const endTime = Date.now()
    const loadTime = endTime - startTime
    
    expect(loadTime).toBeLessThan(2000) // 首页加载应在2秒内
  })
  
  test('scroll performance', async () => {
    // 滚动性能测试
    const scrollCount = 10
    
    for (let i = 0; i < scrollCount; i++) {
      const startTime = Date.now()
      
      // 执行滚动操作
      await driver.swipe(0.5, 0.8, 1000, 0)
      
      const endTime = Date.now()
      const scrollTime = endTime - startTime
      
      expect(scrollTime).toBeLessThan(500) // 每次滚动应在500ms内
    }
  })
})
```

## bm (Build Manager) 构建工具

### 项目构建

```bash
# 清理构建缓存
bm clean

# 编译项目
bm compile

# 构建调试版本
bm build --debug

# 构建发布版本
bm build --release

# 增量编译
bm build --incremental

# 多模块构建
bm build --module entry --module feature1 --module feature2
```

### 代码分析

```bash
# 静态代码分析
bm check

# 代码规范检查
bm lint

# 类型检查
bm typecheck

# 单元测试覆盖率
bm test --coverage
```

### 资源管理

```bash
# 更新资源
bm update-resource

# 压缩图片资源
bm compress-image

# 优化APK包
bm optimize

# 分析资源使用情况
bm analyze-resource
```

### 应用发布

```bash
# 生成发布包
bm pack

# 签名应用
bm sign

# 安装应用
bm install

# 卸载应用
bm uninstall

# 部署到设备
bm deploy -t device_id
```

## app-check 应用合规性检查

### 合规性检查

```bash
# 完整检查
app-check --full

# 快速检查
app-check --quick

# 检查特定模块
app-check --module entry

# 检查权限配置
app-check --permission

# 检查UI组件使用
app-check --ui-component

# 检查性能指标
app-check --performance
```

### 检查项目

```bash
# 检查项目结构
app-check --project

# 检查代码质量
app-check --code-quality

# 检查资源使用
app-check --resource

# 检查网络配置
app-check --network

# 检查安全问题
app-check --security
```

### 生成报告

```bash
# 生成检查报告
app-check --report

# 生成HTML报告
app-check --report --format html

# 生成JSON报告
app-check --report --format json

# 输出到文件
app-check --report --output report.html
```

## binary-sign 应用签名工具

### 基本签名

```bash
# 签名应用
binary-sign app-sign --in /path/to/app.hap --out signed-app.hap

# 使用默认签名
binary-sign app-sign --in /path/to/app.hap

# 指定签名证书
binary-sign app-sign --in /path/to/app.hap --cert certificate.p12

# 使用密码保护的签名
binary-sign app-sign --in /path/to/app.hap --password mypassword
```

### 批量签名

```bash
# 批量签名多个应用
binary-sign batch-sign --config sign-config.json

# 签名配置文件示例：
{
  "apps": [
    {
      "in": "app1.hap",
      "out": "app1-signed.hap"
    },
    {
      "in": "app2.hap",
      "out": "app2-signed.hap"
    }
  ],
  "certificate": "certificate.p12",
  "password": "mypassword"
}
```

### 签名验证

```bash
# 验证签名
binary-sign verify --in /path/to/signed-app.hap

# 查看签名信息
binary-sign info --in /path/to/signed-app.hap

# 检查签名有效期
binary-sign check-expire --in /path/to/signed-app.hap

# 批量验证
binary-sign batch-verify --config verify-config.json
```

### 证书管理

```bash
# 查看可用证书
binary-sign certificate-list

# 导入证书
binary-sign certificate-import --in certificate.p12

# 导出证书
binary-sign certificate-export --out new-certificate.p12

# 设置默认证书
binary-sign certificate-set-default certificate.p12
```

## 最佳实践

### 1. 开发工作流

```bash
# 完整的开发测试流程
# 1. 开发代码
# 2. 本地构建
bm build

# 3. 本地测试
aa test

# 4. 合规性检查
app-check --full

# 5. 应用签名
binary-sign app-sign

# 6. 安装到设备测试
hdc install app -p signed-app.hap

# 7. 日志分析
hdc shell hilog | grep ERROR
```

### 2. 调试技巧

```bash
# 1. 实时监控应用状态
hdc shell aa dump -a

# 2. 监控性能指标
hdc shell top -n 1 | grep app

# 3. 过滤关键日志
hdc shell hilog | grep -E "ERROR|WARN"

# 4. 保存完整日志
hdc shell "logcat -b -v time > app_full.log"

# 5. 远程调试
hdc fport tcp:8000 tcp:8001
```

### 3. 自动化脚本

```bash
# 创建自动化构建脚本
#!/bin/bash

# 构建项目
echo "开始构建项目..."
bm build

# 检查构建结果
if [ $? -eq 0 ]; then
  echo "构建成功"
  
  # 运行测试
  echo "运行测试..."
  aa test
  
  # 检查代码规范
  bm lint
  
  # 合规性检查
  app-check --quick
  
  # 签名应用
  binary-sign app-sign
  
  echo "发布准备完成"
else
  echo "构建失败"
  exit 1
fi
```

### 4. 性能分析

```bash
# 性能分析脚本
#!/bin/bash

echo "=== 性能分析报告 ==="

# 1. CPU使用率
echo "CPU使用率:"
hdc shell top -n 1 | grep -E "CPU|User|System"

# 2. 内存使用
echo -e "\n内存使用:"
hdc shell dumpsys meminfo

# 3. 应用性能
echo -e "\n应用性能:"
hdc shell dumpsys package com.example.app

# 4. 网络状态
echo -e "\n网络状态:"
hdc shell ifconfig -a

# 5. 应用日志
echo -e "\n最近的错误日志:"
hdc shell hilog -l E -t 1000 | head -20
```

### 5. 问题排查

```bash
# 常见问题排查
# 1. 应用启动失败
hdc install app -p app.hap
hdc shell hilog | grep ERROR

# 2. 应用崩溃
hdc shell "logcat -b -v time > crash_log.txt"
bm analyze-crash crash_log.txt

# 3. 应用无响应
hdc shell aa dump -a | grep -i "anr"
bm check --performance

# 4. 网络问题
hdc shell ping -c developer.huawei.com
hdc shell netstat -an | grep -E "ESTABLISHED|TIME_WAIT"

# 5. 存储问题
hdc shell df -h
hdc shell ls -la /data/app/
```

## 集成开发环境

### IDE集成

```bash
# 在DevEco Studio中使用hdc
# 1. 打开Terminal窗口
# 2. 自动连接设备
# 3. 实时查看日志

# 在VS Code中使用hdc
# 1. 安装hdc工具
# 2. 配置环境变量
# 3. 使用扩展或终端执行命令
```

### CI/CD集成

```bash
# GitHub Actions示例
name: Build and Test
on: [push]
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup HarmonyOS SDK
        run: |
          wget https://developer.huawei.com/consumer/cn/resource/sdk/ohos-sdk-tools.tar.gz
          tar -xzf ohos-sdk-tools.tar.gz
      
      - name: Build Project
        run: |
          ./hvigorw build
      
      - name: Run Tests
        run: |
          npm test
      
      - name: App Check
        run: |
          ./app-check --quick
      
      - name: Package App
        run: |
          bm pack
      
      - name: Sign App
        run: |
          binary-sign app-sign --in output/app.hap --cert mycert.p12
      
      - name: Deploy to Device
        run: |
          hdc install app -p output/signed-app.hap
```

## 高级用法

### 多设备操作

```bash
# 列出所有设备
hdc list targets

# 同时在多个设备上安装应用
hdc install app -p app.hap -t device1 -t device2 -t device3

# 在多个设备上同时运行测试
hdc -t device1 test
hdc -t device2 test
hdc -t device3 test

# 从多个设备收集日志
hdc -t device1 shell hilog | grep ERROR > device1.log
hdc -t device2 shell hilog | grep ERROR > device2.log
hdc -t device3 shell hilog | grep ERROR > device3.log
```

### 自动化测试

```bash
# 创建自动化测试脚本
#!/bin/bash

APP_PACKAGE="com.example.app"
TEST_DEVICES=("device1" "device2" "device3")

for device in "${TEST_DEVICES[@]}"; do
  echo "在设备 $device 上运行测试"
  
  # 在指定设备上安装应用
  hdc -t $device install app -p test-app.hap
  
  # 在指定设备上运行测试
  hdc -t $device shell aa test -p $APP_PACKAGE
  
  # 收集日志
  hdc -t $device shell hilog -b > ${device}_test.log
  
  echo "设备 $device 测试完成"
done

echo "所有设备测试完成"
```

## 参考

- [华为官方文档 - hdc命令行工具](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hdc)
- [华为官方文档 - aa自动化测试](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/aa-tool)
- [华为官方文档 - bm构建管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/bm-tool)
- [华为官方文档 - app-check应用检查](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-check-tool)
- [华为官方文档 - binary-sign签名工具](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/binary-sign-tool)
- [华为官方文档 - 命令行工具概览](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/command-line-tools-overview)

> **注意**：不同的工具版本可能有不同的命令语法，请根据实际版本调整命令参数。建议在实际使用前通过 `-h` 或 `--help` 参数查看具体用法。