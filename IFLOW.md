# ijkplayer-android 项目上下文

## 项目概述

**ijkplayer-android** 是一个基于FFmpeg的Android/iOS跨平台视频播放器框架。该项目是Bilibili开源的ijkplayer的一个分支，专门针对Android平台进行了优化和适配。

### 核心特性
- 基于FFmpeg 6.1.1（主分支）和FFmpeg 7.1（实验分支）
- 支持多种视频格式和流媒体协议
- 跨平台支持（Android和iOS）
- 模块化架构设计
- 支持OpenSSL加密流媒体
- 硬件加速解码（MediaCodec）

### 技术栈
- **核心库**: FFmpeg, OpenSSL, libyuv, soundtouch, libsoxr
- **Android构建**: Gradle, CMake, Android NDK
- **跨平台**: C/C++核心代码，Java/Kotlin Android接口
- **脚本语言**: Bash, Python

## 项目结构

```
ijkplayer-android/
├── android/                    # Android平台相关
│   ├── contrib/               # 第三方库构建脚本
│   ├── ijkplayer/             # Android项目主目录
│   │   ├── ijkplayer-example/ # 示例应用
│   │   ├── ijkplayer-java/    # Java层接口
│   │   └── ijkplayer-exo/     # ExoPlayer集成
│   └── patches/               # Android构建补丁
├── cmake/                     # CMake配置
├── config/                    # 构建配置
├── doc/                       # 文档
├── ijkmedia/                  # 核心媒体处理库
│   ├── ijkj4a/               # JNI接口
│   ├── ijkplayer/            # 播放器核心逻辑
│   ├── ijksdl/               # SDL层（音频/视频输出）
│   ├── ijksoundtouch/        # 音频处理
│   └── ijkyuv/               # YUV处理
├── ijkprof/                   # 性能分析工具
├── ios/                       # iOS平台相关
├── tools/                     # 工具脚本
└── extra/                     # 外部依赖（FFmpeg等）
```

## 构建和运行

### 环境要求
- **操作系统**: Ubuntu 22.04 或 macOS
- **Android NDK**: r27（默认，支持arm64-v8a和x86_64，不支持armeabi-v7a）或 r21（支持armeabi-v7a和arm64-v8a）
- **Android Studio**: 2023.1.1 Patch 2
- **Gradle**: 7.2
- **Python**: 3.9.19（通过pyenv安装）
- **其他工具**: git, yasm, ninja-build

### 初始化项目
```bash
# 克隆项目（包含子模块）
git clone https://github.com/ShikinChen/ijkplayer-android --recursive

# 如果需要FFmpeg 7.1分支（API变化较大，稳定待验证）
git clone https://github.com/ShikinChen/ijkplayer-android --recursive -b ijk0.8.8--ff7.1

# 初始化子模块（如果部分模块拉取失败）
cd ijkplayer-android
git submodule update --init --remote --recursive --progress

# 可选：初始化OpenSSL（用于加密流媒体）
./init-android-openssl.sh
```

### 构建FFmpeg
```bash
# 设置NDK路径
export ANDROID_NDK=/path/to/ndk

# 编译FFmpeg（支持arm64, x86_64, 以及NDK r21下的armv7a）
cd android/contrib
./compile-ffmpeg.sh arm64      # 编译arm64版本
./compile-ffmpeg.sh x86_64     # 编译x86_64版本

# 编译armv7a版本（需要NDK r21）
./compile-ffmpeg.sh armv7a
```

### 构建Android库
```bash
# 使用Android Studio导入项目
# 项目路径：android/ijkplayer

# 或者使用Gradle命令行构建
cd android/ijkplayer

# 构建ijkplayer-java模块（生成AAR）
./gradlew :ijkplayer-java:assembleRelease

# 生成的AAR文件位置：
# ijkplayer-java/build/outputs/aar/ijkplayer-java-release.aar
```

### 运行示例应用
```bash
# 在Android Studio中运行ijkplayer-example模块
# 或使用ADB安装示例APK
```

## 开发约定

### 代码结构
1. **C/C++层** (`ijkmedia/`): 核心播放器逻辑，使用FFmpeg API
2. **JNI层** (`ijkmedia/ijkj4a/`): Java Native Interface桥接
3. **Java层** (`android/ijkplayer/ijkplayer-java/`): Android API接口
4. **示例应用** (`android/ijkplayer/ijkplayer-example/`): 使用示例

### 构建配置
- **CMake**: 用于C/C++代码的跨平台构建（CMake 3.22.1+）
- **Gradle**: Android项目构建和依赖管理
- **Bash脚本**: 自动化构建和配置脚本

### 分支管理
- **主分支**: 基于FFmpeg 6.1.1
- **实验分支** (`ijk0.8.8--ff7.1`): 基于FFmpeg 7.1，API变化较大，稳定性待验证

## 常见任务

### 添加新的编解码器支持
1. 修改 `config/module.sh` 中的FFmpeg配置标志
2. 重新编译FFmpeg：`./compile-ffmpeg.sh <arch>`
3. 重新构建Android库

### 调试Native代码
1. 启用LLDB调试：应用相关补丁（`patches/`目录）
2. 在Android Studio中配置Native调试
3. 使用 `ijk-addr2line.sh` 和 `ijk-ndk-stack.sh` 工具

### 性能优化
1. 调整 `config/module-*.sh` 中的编译选项
2. 使用 `ijkprof/` 目录下的性能分析工具
3. 优化CMake构建配置

## 注意事项

1. **NDK版本兼容性**:
   - NDK r27: 支持arm64-v8a和x86_64，不支持armeabi-v7a
   - NDK r21: 支持armeabi-v7a和arm64-v8a
   - MacOS下的NDK 27以下版本在链接SSL库时可能有问题

2. **平台差异**:
   - Ubuntu: 主要开发环境，功能最完整
   - MacOS: 某些功能可能受限（如armeabi-v7a编译）

3. **依赖管理**:
   - 使用git子模块管理FFmpeg等外部依赖
   - 构建前确保所有子模块正确初始化

4. **许可证**:
   - 项目使用Apache 2.0许可证
   - FFmpeg相关代码使用LGPL许可证
   - 注意遵守各组件的许可证要求

## 故障排除

### 常见问题
1. **子模块初始化失败**: 运行 `git submodule update --init --remote --recursive --progress`
2. **FFmpeg编译失败**: 检查NDK路径和环境变量设置
3. **链接错误**: 确保FFmpeg已正确编译并生成静态库
4. **armeabi-v7a不支持**: 切换到NDK r21或使用Ubuntu环境
5. **SSL库链接错误**: 在MacOS上使用NDK r27及以上版本，或在Ubuntu上编译

### 调试工具
- `ijk-addr2line.sh`: 地址转换工具
- `ijk-ndk-stack.sh`: NDK栈跟踪工具
- `patch-debugging-with-lldb.sh`: LLDB调试补丁

## 扩展开发

### 自定义播放器
1. 继承 `tv.danmaku.ijk.media.player.IjkMediaPlayer`
2. 实现自定义的 `IjkMediaPlayer.OnPreparedListener` 等回调
3. 配置播放选项：`IjkMediaPlayer.setOption()`

### 集成到现有项目
1. 将生成的AAR文件添加到项目的 `libs/` 目录
2. 在 `build.gradle` 中添加依赖：
   ```gradle
   implementation files('libs/ijkplayer-java-release.aar')
   ```
3. 按照示例代码初始化和使用播放器

---

*最后更新: 2025-12-10*  
*基于项目分析生成，具体实现细节请参考源代码和文档*