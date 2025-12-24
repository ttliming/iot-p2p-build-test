# .github Directory Documentation

This document provides a comprehensive overview of the `.github` directory structure, contents, and purposes.

## Directory Structure Overview

```
.github/
├── file/                          # Build resource files
│   ├── libs/                      # Pre-compiled libraries
│   │   ├── arm64-v8a/            # ARM64 architecture libraries (5.7M)
│   │   │   └── libcurl.a         # cURL static library
│   │   └── armeabi-v7a/          # ARMv7 architecture libraries (4.3M)
│   │       └── libcurl.a         # cURL static library
│   ├── xp2p_c_demo/              # XP2P C language demo project
│   │   ├── xp2p_c_demo/
│   │   │   └── main.c            # Demo main program
│   │   └── xp2p_c_demo.xcodeproj/ # Xcode project files
│   ├── gradle.properties         # Maven signing configuration template
│   ├── secret.gpg.asc            # GPG encrypted signing key
│   └── secring.gpg.asc           # GPG encrypted keyring
│
├── script/                        # Build scripts for various platforms
│   ├── build_android_combine.sh           # Android combined build script
│   ├── build_enet_android.sh              # Android enet library build
│   ├── build_enet_android_device.sh       # Android device build
│   ├── build_enet_ios.sh                  # iOS platform build script
│   ├── build_enet_linux.sh                # Linux platform build script
│   ├── build_enet_windows.cmd             # Windows build script (64-bit)
│   ├── build_enet_windows32.cmd           # Windows build script (32-bit)
│   ├── update_values_for_sign.sh          # Update signing configuration script
│   └── update_version_for_android.sh      # Android version update script
│
└── workflows/                     # GitHub Actions workflow configurations
    ├── libxp2p_linux.yml                  # Linux CI/CD workflow
    ├── libxp2p_android_combine.yml        # Android combined CI/CD workflow
    ├── libxp2p_windows.yml                # Windows CI/CD workflow
    ├── libxp2p_ios.yml                    # iOS CI/CD workflow
    ├── libxp2p_android.yml.bck            # Android workflow backup
    └── libxp2p_android_device.yml.bck     # Android device workflow backup
```

## 1. Workflows (CI/CD Configurations)

### 1.1 libxp2p_linux.yml - Linux Platform CI/CD

**Trigger Conditions:**
- Push to any branch or tag
- Ignores: Markdown files and LICENSE changes

**Runtime Environment:** ubuntu-24.04

**Main Steps:**
1. Checkout code (fetch-depth: 0 for full history)
2. Install CMake 3.17.0
3. **Debug Build** (triggered by branch push):
   - Clone iot-p2p repository
   - Switch to current branch
   - Build using `build_enet_linux.sh Debug`
4. **Release Build** (triggered by tag push):
   - Clone iot-p2p repository
   - Switch to tag version
   - Build using `build_enet_linux.sh Release`
5. Compress artifacts: `iot-p2p/iot/link/pc_app/p2p_sample` → `xp2p_linux.zip`
6. **Branch Push**: Upload to Artifacts
7. **Tag Push**: Upload to GitHub Release

**Required Secrets:**
- `IOT_GITHUB_ACCESS_TOKEN`
- `GPG_DECRYPT_PASSPHRASE`
- `GITHUB_TOKEN`

---

### 1.2 libxp2p_android_combine.yml - Android Combined Platform CI/CD

**Trigger Conditions:**
- Push to any branch or tag
- Ignores: Markdown files and LICENSE changes

**Runtime Environment:** ubuntu-22.04

**Main Steps:**
1. Checkout code
2. Configure JDK 17 (Temurin distribution)
3. Install CMake 3.17.0
4. Install NDK 25.1.8937393
5. **Debug Build** (branch push):
   - Run `build_android_combine.sh Debug`
   - Build ARM64 and ARMv7 .so libraries
6. **Release Build** (tag push):
   - Run `build_android_combine.sh Release`
7. Compress build artifacts and upload to Artifacts
8. Update Android SDK version number
9. Update Maven signing configuration
10. Decrypt GPG key
11. Build AAR package with Gradle
12. Publish to Maven Central

**Required Secrets:**
- `IOT_SONATYPE_USERNAME` - Maven Central username
- `IOT_SONATYPE_PASSWORD` - Maven Central password
- `GPG_DECRYPT_PASSPHRASE` - GPG decryption password
- `IOT_GPG_KEYNAME` - GPG key ID
- `IOT_GPG_PASSPHRASE` - GPG signing password
- `IOT_GITHUB_ACCESS_TOKEN` - GitHub access token

**Artifacts:**
- xp2p_artifacts.zip (contains compiled .so libraries)
- AAR package published to Maven Central

---

### 1.3 libxp2p_windows.yml - Windows Platform CI/CD

**Trigger Conditions:**
- Push to any branch or tag
- Ignores: Markdown files and LICENSE changes

**Runtime Environment:** windows-2022

**Main Steps:**
1. Checkout code
2. Install CMake 3.21.0
3. **Debug Build** (branch push):
   - Build 32-bit version using `build_enet_windows32.cmd Debug`
4. **Release Build** (tag push):
   - Build 64-bit version using `build_enet_windows.cmd Release`
5. Compress artifacts: `iot-p2p/iot/link/pc_app/p2p_sample` → `xp2p_windows.zip`
6. **Branch Push**: Upload to Artifacts (named xp2p_windows32.zip)
7. **Tag Push**: Upload to GitHub Release

**Required Secrets:**
- `IOT_GITHUB_ACCESS_TOKEN`
- `GPG_DECRYPT_PASSPHRASE`
- `GITHUB_TOKEN`

---

### 1.4 libxp2p_ios.yml - iOS Platform CI/CD

**Trigger Conditions:**
- Push to any branch or tag
- Ignores: Markdown files and LICENSE changes

**Runtime Environment:** macos-latest

**Main Steps:**
1. Checkout code
2. Install CMake 3.17.0
3. List available Xcode versions
4. Select Xcode 16 and verify SDK
5. **Debug Build** (branch push):
   - Build using `build_enet_ios.sh Debug`
6. **Release Build** (tag push):
   - Build using `build_enet_ios.sh Release`
7. Upload build artifact: `iot-p2p/build/ios/Release-iphoneos/libenet.a`

**Required Secrets:**
- `IOT_GITHUB_ACCESS_TOKEN`
- `GPG_DECRYPT_PASSPHRASE`

**Artifacts:**
- libenet_ios.a (static library)

---

### 1.5 Backup Workflow Files

- **libxp2p_android.yml.bck** - Old Android workflow (uses ubuntu-18.04)
- **libxp2p_android_device.yml.bck** - Android device-specific workflow backup

These files are kept for reference and are not currently active.

---

## 2. Scripts (Build Scripts)

### 2.1 build_enet_linux.sh - Linux Build Script

**Functionality:**
- Clone iot-p2p repository
- Switch branch or tag based on build type (Debug/Release)
- Update version number in source code
- Build enet library using CMake (Linux platform)
- Compile app_interface library and sample application

**Parameters:**
- `$1`: Build type (Debug or Release)

**Environment Variables:**
- `GIT_BRANCH_IMAGE_VERSION`: Git branch/tag name
- `GIT_ACCESS_TOKEN`: GitHub access token

**Output:**
- `iot-p2p/iot/link/pc_app/p2p_sample/` - Compiled sample application

---

### 2.2 build_android_combine.sh - Android Combined Build Script

**Functionality:**
- Clone iot-p2p repository
- Copy app_interface source files to Android project
- Update P2P code version number
- Copy libcurl.a library files
- Build ARM64 and ARMv7 .so libraries using CMake and NDK
- Move compiled library files to correct locations
- Execute Android project's cmake_build.sh

**Parameters:**
- `$1`: Build type (Debug or Release)

**NDK Version Used:** 25.1.8937393

**Output:**
- `iot-p2p/iot/device/android_device/device_video_aar/explorer-app-video-sdk/libs/` - Compiled .so files

---

### 2.3 build_enet_ios.sh - iOS Build Script

**Functionality:**
- Clone iot-p2p repository
- Switch branch/tag based on build type
- Update version number
- Configure iOS platform build using CMake
- Generate static library libenet.a

**Parameters:**
- `$1`: Build type (Debug or Release)

**Output:**
- `iot-p2p/build/ios/Release-iphoneos/libenet.a`

---

### 2.4 build_enet_windows.cmd / build_enet_windows32.cmd - Windows Build Scripts

**Functionality:**
- Clone iot-p2p repository
- Configure Windows platform build using CMake
- Build enet library and sample application

**Parameters:**
- `$1`: Build type (Debug or Release)
- `$2`: Branch name

**Differences:**
- `build_enet_windows.cmd` - Builds 64-bit version
- `build_enet_windows32.cmd` - Builds 32-bit version

---

### 2.5 update_version_for_android.sh - Android Version Update Script

**Functionality:**
- Automatically calculate and update Android library version number
- Debug mode: Use latest tag+1 to generate SNAPSHOT version
- Release mode: Use current tag as version number

**Parameters:**
- `$1`: Build type (Debug or Release)
- `$2`: build.gradle file path

**Version Rules:**
- Debug: `{latest_tag+1}-SNAPSHOT`
- Release: `{current_tag}`

---

### 2.6 update_values_for_sign.sh - Signing Configuration Update Script

**Functionality:**
- Update Maven signing configuration in gradle.properties
- Replace placeholders with actual key information

**Replaced Placeholders:**
- `MY_KEY_ID` → GPG key ID
- `MY_PASSWORD` → GPG password
- `MY_KEY_RING_FILE` → GPG keyring file path
- `MY_MAVEN_USERNAME` → Maven Central username
- `MY_MAVEN_PASSWORD` → Maven Central password

---

### 2.7 Other Build Scripts

- **build_enet_android.sh** - Early Android build script
- **build_enet_android_device.sh** - Android device-specific build script

---

## 3. File (Resource Files)

### 3.1 libs/ - Pre-compiled Library Files

Contains libcurl static libraries for Android platforms:
- **arm64-v8a/libcurl.a** (5.7M) - 64-bit ARM architecture
- **armeabi-v7a/libcurl.a** (4.3M) - 32-bit ARM architecture

These libraries are copied to appropriate directories during Android build process.

---

### 3.2 xp2p_c_demo/ - C Language Demo Project

**Contents:**
- Xcode project files
- main.c - Basic example code

**Purpose:**
- Demonstrates how to use XP2P C API
- Serves as integration reference

---

### 3.3 Encrypted Files

- **secret.gpg.asc** (6.7K) - GPG encrypted Maven signing key
- **secring.gpg.asc** (3.7K) - GPG encrypted keyring (backup)

**Purpose:**
In CI/CD pipelines, these files are decrypted using `GPG_DECRYPT_PASSPHRASE` for signing AAR packages published to Maven Central.

---

### 3.4 gradle.properties

Maven publishing configuration template with placeholders:
```properties
signing.keyId=MY_KEY_ID
signing.password=MY_PASSWORD
signing.secretKeyRingFile=MY_KEY_RING_FILE
```

Populated with actual values during build by `update_values_for_sign.sh` script.

---

## 4. Workflow Processes

### 4.1 Development Flow (Branch Push)

1. Developer pushes code to branch
2. Triggers corresponding platform CI workflows
3. Executes Debug build
4. Uploads build artifacts to GitHub Artifacts
5. Developer can download Artifacts for testing

### 4.2 Release Flow (Tag Push)

1. Create version tag and push
2. Triggers all platform CI workflows
3. Executes Release build
4. **Linux/Windows**: Upload binaries to GitHub Release
5. **iOS**: Upload libenet.a to Artifacts
6. **Android**: Publish AAR to Maven Central

### 4.3 Android Special Flow

Android platform has additional steps:
1. Build .so libraries for both ARM64 and ARMv7 architectures
2. Package into AAR
3. Sign with GPG
4. Publish to Maven Central repository

---

## 5. Technologies and Tools Used

### 5.1 Build Tools
- **CMake**: Cross-platform build system (versions 3.17.0-3.21.0)
- **Gradle**: Android project build
- **Xcode**: iOS project build
- **NDK**: Android Native Development Kit (version 25.1.8937393)

### 5.2 CI/CD
- **GitHub Actions**: Automated build and deployment
- **actions/checkout@v2**: Code checkout
- **jwlawson/actions-setup-cmake**: CMake installation
- **actions/setup-java@v3**: Java environment configuration
- **actions/upload-artifact@v4**: Artifact upload
- **svenstaro/upload-release-action@v2**: Release upload

### 5.3 Publishing and Signing
- **Maven Central**: Java/Android library publishing platform
- **GPG**: Used for package signature verification

---

## 6. Security Best Practices

### 6.1 Secrets Management

All sensitive information is stored in GitHub Secrets:
- GitHub access tokens
- Maven Central credentials
- GPG keys and passwords

### 6.2 Key File Encryption

GPG key files are stored encrypted in the repository using symmetric encryption, only decrypted in CI environment.

---

## 7. Maintenance Recommendations

### 7.1 Regular Updates

- Regularly update GitHub Actions versions
- Update CMake, NDK and other tool versions
- Check for security updates in dependencies

### 7.2 Cleanup Backup Files

Consider removing or archiving `.bck` backup files if no longer needed.

### 7.3 Documentation Maintenance

Update this documentation promptly when workflows or scripts change.

---

## 8. Frequently Asked Questions

### Q1: How to trigger builds for specific platforms?
A: All platforms are triggered on every push. To build only specific platforms, modify the trigger conditions in workflows.

### Q2: How to view build logs?
A: View all workflow run records and detailed logs in the "Actions" tab of the GitHub repository.

### Q3: What's the difference between Debug and Release builds?
A: Debug builds from branches generate versions with debug symbols; Release builds from tags are optimized and may be published to public repositories.

### Q4: How to add support for new platforms?
A: 
1. Add corresponding build script in `script/` directory
2. Add corresponding workflow configuration in `workflows/` directory
3. Ensure required dependency files are placed in `file/` directory

---

## 9. Related Resources

- **Main Repository**: https://github.com/tencentyun/iot-p2p
- **GitHub Actions Documentation**: https://docs.github.com/actions
- **CMake Documentation**: https://cmake.org/documentation/
- **Maven Central**: https://central.sonatype.com/

---

**Last Updated**: 2025-12-24
**Maintainer**: IoT P2P Team
