# .github 目录说明文档

本文档详细说明了 `.github` 目录的结构、内容和用途。

## 目录结构概览

```
.github/
├── file/                          # 构建所需的资源文件
│   ├── libs/                      # 预编译的库文件
│   │   ├── arm64-v8a/            # ARM64架构的库 (5.7M)
│   │   │   └── libcurl.a         # cURL静态库
│   │   └── armeabi-v7a/          # ARMv7架构的库 (4.3M)
│   │       └── libcurl.a         # cURL静态库
│   ├── xp2p_c_demo/              # XP2P C语言示例项目
│   │   ├── xp2p_c_demo/
│   │   │   └── main.c            # 示例主程序
│   │   └── xp2p_c_demo.xcodeproj/ # Xcode项目文件
│   ├── gradle.properties         # Maven签名配置模板
│   ├── secret.gpg.asc            # GPG加密的签名密钥
│   └── secring.gpg.asc           # GPG加密的密钥环
│
├── script/                        # 各平台构建脚本
│   ├── build_android_combine.sh           # Android综合构建脚本
│   ├── build_enet_android.sh              # Android enet库构建
│   ├── build_enet_android_device.sh       # Android设备端构建
│   ├── build_enet_ios.sh                  # iOS平台构建脚本
│   ├── build_enet_linux.sh                # Linux平台构建脚本
│   ├── build_enet_windows.cmd             # Windows构建脚本 (64位)
│   ├── build_enet_windows32.cmd           # Windows构建脚本 (32位)
│   ├── update_values_for_sign.sh          # 更新签名配置脚本
│   └── update_version_for_android.sh      # Android版本号更新脚本
│
└── workflows/                     # GitHub Actions工作流配置
    ├── libxp2p_linux.yml                  # Linux CI/CD工作流
    ├── libxp2p_android_combine.yml        # Android综合CI/CD工作流
    ├── libxp2p_windows.yml                # Windows CI/CD工作流
    ├── libxp2p_ios.yml                    # iOS CI/CD工作流
    ├── libxp2p_android.yml.bck            # Android工作流备份
    └── libxp2p_android_device.yml.bck     # Android设备工作流备份
```

## 1. Workflows（工作流配置）

### 1.1 libxp2p_linux.yml - Linux平台CI/CD

**触发条件：**
- Push到任意分支或标签
- 忽略：Markdown文件和LICENSE的变更

**运行环境：** ubuntu-24.04

**主要步骤：**
1. 检出代码（fetch-depth: 0，获取完整历史）
2. 安装CMake 3.17.0
3. **Debug构建**（分支推送触发）：
   - 克隆iot-p2p仓库
   - 切换到当前分支
   - 使用`build_enet_linux.sh Debug`构建
4. **Release构建**（标签推送触发）：
   - 克隆iot-p2p仓库
   - 切换到标签版本
   - 使用`build_enet_linux.sh Release`构建
5. 压缩产物：`iot-p2p/iot/link/pc_app/p2p_sample` → `xp2p_linux.zip`
6. **分支推送**：上传到Artifacts
7. **标签推送**：上传到GitHub Release

**使用的Secrets：**
- `IOT_GITHUB_ACCESS_TOKEN`
- `GPG_DECRYPT_PASSPHRASE`
- `GITHUB_TOKEN`

---

### 1.2 libxp2p_android_combine.yml - Android综合平台CI/CD

**触发条件：**
- Push到任意分支或标签
- 忽略：Markdown文件和LICENSE的变更

**运行环境：** ubuntu-22.04

**主要步骤：**
1. 检出代码
2. 配置JDK 17（Temurin发行版）
3. 安装CMake 3.17.0
4. 安装NDK 25.1.8937393
5. **Debug构建**（分支推送）：
   - 运行`build_android_combine.sh Debug`
   - 构建ARM64和ARMv7架构的.so库
6. **Release构建**（标签推送）：
   - 运行`build_android_combine.sh Release`
7. 压缩构建产物并上传到Artifacts
8. 更新Android SDK版本号
9. 更新Maven签名配置
10. 解密GPG密钥
11. 使用Gradle构建AAR包
12. 发布到Maven Central

**使用的Secrets：**
- `IOT_SONATYPE_USERNAME` - Maven Central用户名
- `IOT_SONATYPE_PASSWORD` - Maven Central密码
- `GPG_DECRYPT_PASSPHRASE` - GPG解密密码
- `IOT_GPG_KEYNAME` - GPG密钥ID
- `IOT_GPG_PASSPHRASE` - GPG签名密码
- `IOT_GITHUB_ACCESS_TOKEN` - GitHub访问令牌

**产物：**
- xp2p_artifacts.zip（包含编译的.so库）
- 发布到Maven Central的AAR包

---

### 1.3 libxp2p_windows.yml - Windows平台CI/CD

**触发条件：**
- Push到任意分支或标签
- 忽略：Markdown文件和LICENSE的变更

**运行环境：** windows-2022

**主要步骤：**
1. 检出代码
2. 安装CMake 3.21.0
3. **Debug构建**（分支推送）：
   - 使用`build_enet_windows32.cmd Debug`构建32位版本
4. **Release构建**（标签推送）：
   - 使用`build_enet_windows.cmd Release`构建64位版本
5. 压缩产物：`iot-p2p/iot/link/pc_app/p2p_sample` → `xp2p_windows.zip`
6. **分支推送**：上传到Artifacts（命名为xp2p_windows32.zip）
7. **标签推送**：上传到GitHub Release

**使用的Secrets：**
- `IOT_GITHUB_ACCESS_TOKEN`
- `GPG_DECRYPT_PASSPHRASE`
- `GITHUB_TOKEN`

---

### 1.4 libxp2p_ios.yml - iOS平台CI/CD

**触发条件：**
- Push到任意分支或标签
- 忽略：Markdown文件和LICENSE的变更

**运行环境：** macos-latest

**主要步骤：**
1. 检出代码
2. 安装CMake 3.17.0
3. 列出可用的Xcode版本
4. 选择Xcode 16并验证SDK
5. **Debug构建**（分支推送）：
   - 使用`build_enet_ios.sh Debug`构建
6. **Release构建**（标签推送）：
   - 使用`build_enet_ios.sh Release`构建
7. 上传构建产物：`iot-p2p/build/ios/Release-iphoneos/libenet.a`

**使用的Secrets：**
- `IOT_GITHUB_ACCESS_TOKEN`
- `GPG_DECRYPT_PASSPHRASE`

**产物：**
- libenet_ios.a（静态库）

---

### 1.5 备份的工作流文件

- **libxp2p_android.yml.bck** - 旧版Android工作流（使用ubuntu-18.04）
- **libxp2p_android_device.yml.bck** - Android设备端专用工作流备份

这些文件被保留作为参考，当前未激活使用。

---

## 2. Scripts（构建脚本）

### 2.1 build_enet_linux.sh - Linux构建脚本

**功能：**
- 克隆iot-p2p仓库
- 根据构建类型（Debug/Release）切换分支或标签
- 更新版本号到源代码中
- 使用CMake构建enet库（Linux平台）
- 编译app_interface库和示例应用

**参数：**
- `$1`: 构建类型（Debug或Release）

**环境变量：**
- `GIT_BRANCH_IMAGE_VERSION`: Git分支/标签名称
- `GIT_ACCESS_TOKEN`: GitHub访问令牌

**输出：**
- `iot-p2p/iot/link/pc_app/p2p_sample/` - 编译好的示例应用

---

### 2.2 build_android_combine.sh - Android综合构建脚本

**功能：**
- 克隆iot-p2p仓库
- 拷贝app_interface源文件到Android项目
- 更新P2P代码版本号
- 拷贝libcurl.a库文件
- 使用CMake和NDK构建ARM64和ARMv7架构的.so库
- 移动编译的库文件到正确位置
- 执行Android项目的cmake_build.sh

**参数：**
- `$1`: 构建类型（Debug或Release）

**使用的NDK版本：** 25.1.8937393

**输出：**
- `iot-p2p/iot/device/android_device/device_video_aar/explorer-app-video-sdk/libs/` - 编译的.so文件

---

### 2.3 build_enet_ios.sh - iOS构建脚本

**功能：**
- 克隆iot-p2p仓库
- 根据构建类型切换分支/标签
- 更新版本号
- 使用CMake配置iOS平台构建
- 生成静态库libenet.a

**参数：**
- `$1`: 构建类型（Debug或Release）

**输出：**
- `iot-p2p/build/ios/Release-iphoneos/libenet.a`

---

### 2.4 build_enet_windows.cmd / build_enet_windows32.cmd - Windows构建脚本

**功能：**
- 克隆iot-p2p仓库
- 使用CMake配置Windows平台构建
- 构建enet库和示例应用

**参数：**
- `$1`: 构建类型（Debug或Release）
- `$2`: 分支名称

**区别：**
- `build_enet_windows.cmd` - 构建64位版本
- `build_enet_windows32.cmd` - 构建32位版本

---

### 2.5 update_version_for_android.sh - Android版本更新脚本

**功能：**
- 自动计算并更新Android库的版本号
- Debug模式：使用最新tag+1生成SNAPSHOT版本
- Release模式：使用当前tag作为版本号

**参数：**
- `$1`: 构建类型（Debug或Release）
- `$2`: build.gradle文件路径

**版本规则：**
- Debug: `{最新tag+1}-SNAPSHOT`
- Release: `{当前tag}`

---

### 2.6 update_values_for_sign.sh - 签名配置更新脚本

**功能：**
- 更新gradle.properties中的Maven签名配置
- 替换占位符为实际的密钥信息

**替换的占位符：**
- `MY_KEY_ID` → GPG密钥ID
- `MY_PASSWORD` → GPG密码
- `MY_KEY_RING_FILE` → GPG密钥环文件路径
- `MY_MAVEN_USERNAME` → Maven Central用户名
- `MY_MAVEN_PASSWORD` → Maven Central密码

---

### 2.7 其他构建脚本

- **build_enet_android.sh** - 早期的Android构建脚本
- **build_enet_android_device.sh** - Android设备端专用构建脚本

---

## 3. File（资源文件）

### 3.1 libs/ - 预编译库文件

包含Android平台的libcurl静态库：
- **arm64-v8a/libcurl.a** (5.7M) - 64位ARM架构
- **armeabi-v7a/libcurl.a** (4.3M) - 32位ARM架构

这些库在Android构建过程中被拷贝到相应的目录。

---

### 3.2 xp2p_c_demo/ - C语言示例项目

**内容：**
- Xcode项目文件
- main.c - 基础示例代码

**用途：**
- 展示如何使用XP2P C API
- 作为集成参考

---

### 3.3 加密文件

- **secret.gpg.asc** (6.7K) - GPG加密的Maven签名密钥
- **secring.gpg.asc** (3.7K) - GPG加密的密钥环（备份）

**用途：**
在CI/CD流程中，这些文件使用`GPG_DECRYPT_PASSPHRASE`解密，用于签名发布到Maven Central的AAR包。

---

### 3.4 gradle.properties

Maven发布配置模板，包含占位符：
```properties
signing.keyId=MY_KEY_ID
signing.password=MY_PASSWORD
signing.secretKeyRingFile=MY_KEY_RING_FILE
```

在构建过程中由`update_values_for_sign.sh`脚本填充实际值。

---

## 4. 工作流程说明

### 4.1 开发流程（分支推送）

1. 开发者推送代码到分支
2. 触发相应平台的CI工作流
3. 执行Debug构建
4. 上传构建产物到GitHub Artifacts
5. 开发者可以下载Artifacts进行测试

### 4.2 发布流程（标签推送）

1. 创建版本标签并推送
2. 触发所有平台的CI工作流
3. 执行Release构建
4. **Linux/Windows**: 上传二进制文件到GitHub Release
5. **iOS**: 上传libenet.a到Artifacts
6. **Android**: 发布AAR到Maven Central

### 4.3 Android特殊流程

Android平台有额外的步骤：
1. 构建ARM64和ARMv7两个架构的.so库
2. 打包成AAR
3. 使用GPG签名
4. 发布到Maven Central仓库

---

## 5. 使用的技术和工具

### 5.1 构建工具
- **CMake**: 跨平台构建系统（版本3.17.0-3.21.0）
- **Gradle**: Android项目构建
- **Xcode**: iOS项目构建
- **NDK**: Android Native开发套件（版本25.1.8937393）

### 5.2 CI/CD
- **GitHub Actions**: 自动化构建和部署
- **actions/checkout@v2**: 代码检出
- **jwlawson/actions-setup-cmake**: CMake安装
- **actions/setup-java@v3**: Java环境配置
- **actions/upload-artifact@v4**: 产物上传
- **svenstaro/upload-release-action@v2**: Release上传

### 5.3 发布和签名
- **Maven Central**: Java/Android库发布平台
- **GPG**: 用于包签名验证

---

## 6. 安全最佳实践

### 6.1 Secrets管理

所有敏感信息都存储在GitHub Secrets中：
- GitHub访问令牌
- Maven Central凭据
- GPG密钥和密码

### 6.2 密钥文件加密

GPG密钥文件使用对称加密存储在仓库中，只在CI环境中解密使用。

---

## 7. 维护建议

### 7.1 定期更新

- 定期更新GitHub Actions版本
- 更新CMake、NDK等工具版本
- 检查依赖库的安全更新

### 7.2 清理备份文件

考虑删除或归档`.bck`备份文件，如果不再需要。

### 7.3 文档维护

当工作流或脚本变更时，及时更新本文档。

---

## 8. 常见问题

### Q1: 如何触发特定平台的构建？
A: 所有平台在每次push时都会触发。如果只想构建特定平台，可以修改工作流的触发条件。

### Q2: 如何查看构建日志？
A: 在GitHub仓库的"Actions"标签页中可以查看所有工作流运行记录和详细日志。

### Q3: Debug和Release构建的区别？
A: Debug构建从分支构建，生成带调试符号的版本；Release构建从标签构建，进行优化并可能发布到公共仓库。

### Q4: 如何添加新的平台支持？
A: 
1. 在`script/`目录添加对应的构建脚本
2. 在`workflows/`目录添加对应的工作流配置
3. 确保所需的依赖文件放在`file/`目录

---

## 9. 相关资源

- **主仓库**: https://github.com/tencentyun/iot-p2p
- **GitHub Actions文档**: https://docs.github.com/actions
- **CMake文档**: https://cmake.org/documentation/
- **Maven Central**: https://central.sonatype.com/

---

**文档最后更新**: 2025-12-24
**维护者**: IoT P2P团队
