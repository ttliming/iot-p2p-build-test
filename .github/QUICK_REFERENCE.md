# .github 目录快速参考指南

## 快速导航

### 工作流文件
| 文件 | 平台 | 运行环境 | 用途 |
|------|------|----------|------|
| `workflows/libxp2p_linux.yml` | Linux | ubuntu-24.04 | 构建Linux平台的XP2P库 |
| `workflows/libxp2p_android_combine.yml` | Android | ubuntu-22.04 | 构建Android平台的.so库并发布到Maven |
| `workflows/libxp2p_windows.yml` | Windows | windows-2022 | 构建Windows平台的XP2P库 |
| `workflows/libxp2p_ios.yml` | iOS | macos-latest | 构建iOS平台的静态库 |

### 构建脚本
| 脚本 | 平台 | 功能 |
|------|------|------|
| `script/build_enet_linux.sh` | Linux | 克隆仓库、构建enet和示例应用 |
| `script/build_android_combine.sh` | Android | 构建ARM64和ARMv7架构的.so库 |
| `script/build_enet_ios.sh` | iOS | 构建iOS静态库libenet.a |
| `script/build_enet_windows.cmd` | Windows | 构建64位Windows版本 |
| `script/build_enet_windows32.cmd` | Windows | 构建32位Windows版本 |
| `script/update_version_for_android.sh` | Android | 自动更新版本号 |
| `script/update_values_for_sign.sh` | Android | 更新Maven签名配置 |

### 资源文件
| 文件/目录 | 大小 | 用途 |
|-----------|------|------|
| `file/libs/arm64-v8a/libcurl.a` | 5.7M | Android ARM64架构的cURL库 |
| `file/libs/armeabi-v7a/libcurl.a` | 4.3M | Android ARMv7架构的cURL库 |
| `file/secret.gpg.asc` | 6.7K | GPG加密的Maven签名密钥 |
| `file/gradle.properties` | 95B | Maven签名配置模板 |
| `file/xp2p_c_demo/` | - | XP2P C语言示例项目 |

## 常用命令

### 本地测试构建脚本
```bash
# Linux
GIT_BRANCH_IMAGE_VERSION=main GIT_ACCESS_TOKEN=xxx sh .github/script/build_enet_linux.sh Debug

# Android
GIT_BRANCH_IMAGE_VERSION=main GIT_ACCESS_TOKEN=xxx sh .github/script/build_android_combine.sh Debug

# iOS
GIT_BRANCH_IMAGE_VERSION=main GIT_ACCESS_TOKEN=xxx sh .github/script/build_enet_ios.sh Debug

# Windows (PowerShell)
.\\.github\\script\\build_enet_windows32.cmd Debug main
```

### 查看工作流运行状态
```bash
# 使用 GitHub CLI
gh run list
gh run view <run-id>
gh run view <run-id> --log
```

## 触发机制

### 自动触发
- **分支推送**: 触发Debug构建，产物上传到Artifacts
- **标签推送**: 触发Release构建，产物上传到Release

### 忽略触发
以下文件变更不会触发构建：
- `*.md` (Markdown文件)
- `LICENSE`

## 版本号规则

### Android
- **Debug**: `{最新tag+1}-SNAPSHOT` (例如: `1.0.1-SNAPSHOT`)
- **Release**: `{当前tag}` (例如: `v1.0.0`)

### 其他平台
版本号直接嵌入到代码中的 `VIDEOSDKVERSION` 常量

## 产物输出

### Linux
- **位置**: `iot-p2p/iot/link/pc_app/p2p_sample/`
- **格式**: `xp2p_linux.zip`

### Android
- **位置**: 
  - `.so文件`: `iot-p2p/iot/device/android_device/device_video_aar/explorer-app-video-sdk/libs/`
  - `AAR包`: 发布到Maven Central
- **格式**: 
  - `xp2p_artifacts.zip` (Artifacts)
  - `explorer-app-video-sdk-{version}.aar` (Maven)

### Windows
- **位置**: `iot-p2p/iot/link/pc_app/p2p_sample/`
- **格式**: `xp2p_windows.zip`

### iOS
- **位置**: `iot-p2p/build/ios/Release-iphoneos/`
- **格式**: `libenet.a`

## 必需的Secrets

| Secret名称 | 用途 | 使用平台 |
|-----------|------|----------|
| `IOT_GITHUB_ACCESS_TOKEN` | 访问私有仓库 | 全部 |
| `GPG_DECRYPT_PASSPHRASE` | 解密GPG密钥 | 全部 |
| `IOT_SONATYPE_USERNAME` | Maven Central用户名 | Android |
| `IOT_SONATYPE_PASSWORD` | Maven Central密码 | Android |
| `IOT_GPG_KEYNAME` | GPG密钥ID | Android |
| `IOT_GPG_PASSPHRASE` | GPG签名密码 | Android |
| `GITHUB_TOKEN` | GitHub API访问 | 自动提供 |

## 依赖工具版本

| 工具 | 版本 | 平台 |
|------|------|------|
| CMake | 3.17.0 | Linux, Android, iOS |
| CMake | 3.21.0 | Windows |
| JDK | 17 (Temurin) | Android |
| NDK | 25.1.8937393 | Android |
| Xcode | 16 | iOS |

## 故障排查

### 问题：工作流没有触发
- 检查是否修改了忽略的文件（.md, LICENSE）
- 确认推送到了正确的分支或创建了标签

### 问题：构建失败
1. 查看Actions页面的详细日志
2. 检查Secrets是否配置正确
3. 验证依赖工具版本是否匹配

### 问题：Android发布失败
- 确认Maven Central凭据正确
- 检查GPG密钥是否过期
- 验证版本号格式是否正确

### 问题：找不到产物
- **分支推送**: 在Actions页面的Artifacts中下载
- **标签推送**: 在Releases页面查找

## 开发者工作流

### 日常开发
1. 在功能分支上开发
2. 推送到GitHub触发Debug构建
3. 从Artifacts下载产物进行测试
4. 合并到主分支

### 发布版本
1. 确认所有更改已合并到主分支
2. 创建版本标签: `git tag v1.0.0`
3. 推送标签: `git push origin v1.0.0`
4. 等待所有平台构建完成
5. 验证Release页面的产物
6. 对于Android，验证Maven Central上的发布

## 联系方式

如有问题，请：
1. 查看完整文档: [README.md](README.md)
2. 提交Issue到项目仓库
3. 联系IoT P2P团队

---

**快速参考指南版本**: 1.0  
**最后更新**: 2025-12-24
