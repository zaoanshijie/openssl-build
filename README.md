# OpenSSL Build

使用 GitHub Actions 自动编译 OpenSSL，支持 Windows / Linux 多平台、多编译器。

## 编译配置

| 组 | 编译器 | 架构 | 目标平台 | Runner |
|----|--------|------|----------|--------|
| windows-msvc | Visual Studio 2022 (MSVC) | x64 | Windows | windows-latest |
| windows-llvm-mingw | LLVM-MinGW (Clang) | x64 | Windows (交叉编译) | ubuntu-latest |
| ubuntu-gcc-x86_64 | GCC | x86_64 | Linux | ubuntu-latest |
| ubuntu-gcc-aarch64 | GCC | aarch64 | Linux | ubuntu-24.04-arm |
| ubuntu-clang-x86_64 | Clang | x86_64 | Linux | ubuntu-latest |
| ubuntu-clang-aarch64 | Clang | aarch64 | Linux | ubuntu-24.04-arm |

## 编译特性

- **zlib** 静态链接压缩支持
- **RC5** 加密算法
- **MD2** 哈希算法
- **弱 SSL 密码** (RC4 等)

## 版本策略

- 每日 `UTC 1:00` 自动检测 OpenSSL 最新版本
- 固定编译 `openssl-3.0.22` 和 `openssl-3.6.4`
- 同时编译最新稳定版和绝对最新版（含预发布）
- 预发布版本自动标记为 GitHub Prerelease

## 工作流

```
Detect Versions → Build Matrix (版本 × 配置) → Create Release
```

每个平台执行：`Configure → Build (make/nmake) → Test → Install → Verify → Upload Artifact`

Release 阶段下载同版本所有平台的 Artifacts，打包为 `.tar.gz` 并上传到 GitHub Release，已存在的 Release 自动跳过。

## Release 下载

构建产物发布在 [Releases](https://github.com/zaoanshijie/openssl-build/releases) 页面。

## 手动触发

在 [Actions](https://github.com/zaoanshijie/openssl-build/actions) 页面选择 **Build OpenSSL** → **Run workflow**。