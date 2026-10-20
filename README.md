# OpenSSL Build

使用 GitHub Actions 自动编译 OpenSSL，支持 Windows / Linux 多平台、多编译器。

## 编译配置

| 组 | 编译器 | 架构 | Runner | glibc 基线 |
|----|--------|------|--------|-----------|
| windows-msvc | Visual Studio 2022 (MSVC) | x64 | windows-2025 | - |
| windows-llvm-mingw | LLVM-MinGW (Clang) | x64 | ubuntu-24.04 | - |
| ubuntu-24.04-gcc-x86_64 | GCC 13 | x86_64 | ubuntu-24.04 | >= 2.39 |
| ubuntu-24.04-gcc-aarch64 | GCC 13 | aarch64 | ubuntu-24.04-arm | >= 2.39 |
| ubuntu-24.04-clang-x86_64 | Clang 18 | x86_64 | ubuntu-24.04 | >= 2.39 |
| ubuntu-24.04-clang-aarch64 | Clang 18 | aarch64 | ubuntu-24.04-arm | >= 2.39 |
| ubuntu-22.04-gcc-x86_64 | GCC 11 | x86_64 | ubuntu-22.04 | >= 2.35 |
| ubuntu-22.04-gcc-aarch64 | GCC 11 | aarch64 | ubuntu-22.04-arm | >= 2.35 |

### 该选哪个 Linux 产物？

- **Ubuntu 24.04 产物**：编译器 / glibc 更新，性能与优化更好，适用于 Ubuntu 24.04+、Debian 13+ 以及各滚动发行版（Fedora、Arch 等）。
- **Ubuntu 22.04 产物**：glibc 基线更低（2.35），可向下兼容 Ubuntu 22.04、Debian 12、CentOS Stream 9 / RHEL 9 等旧系统；在更高版本系统上同样可直接运行。

> 链接了 glibc 的二进制无法在低于编译时 glibc 版本的系统上运行，需要覆盖旧系统时优先用 22.04 产物。

每个产物包内的 `BUILD_INFO.txt` 记录了实际使用的 runner、编译器版本、glibc 版本与 OpenSSL 版本，可直接据此判断兼容性。

## Runner 版本策略

所有任务均**显式钉住** runner 镜像版本，不使用 `ubuntu-latest` / `windows-latest`：

- `ubuntu-latest` 自 2026-10-19 起陆续从 Ubuntu 24.04 迁移到 26.04，同一份 workflow 在不同时间会落到不同镜像，导致产物的编译器与 glibc 版本发生漂移。
- 因此固定为 `ubuntu-24.04` / `ubuntu-24.04-arm`，另加 `ubuntu-22.04` / `ubuntu-22.04-arm` 作为低 glibc 基线产物。
- 升级到 Ubuntu 26.04 时，只需把对应条目的 `os` 改为 `ubuntu-26.04` / `ubuntu-26.04-arm`，并新增一组配置名，避免覆盖已有产物。

## 编译特性

- **zlib** 静态链接压缩支持
- **RC5** 加密算法
- **MD2** 哈希算法
- **弱 SSL 密码** (RC4 等)

## 版本策略

- 每日 `UTC 1:00` 自动检测 OpenSSL 最新版本
- 固定编译 `openssl-3.0.22` 和 `openssl-3.6.4`
- 同时编译最新稳定版和绝对最新版（含预发布）
- **产物是否为 Prerelease 完全跟随上游**：查询 `openssl/openssl` 对该 tag 的 Release 是否标记 prerelease，上游是正式版则产物也是正式版，上游是预发布则产物也是预发布（上游没有对应 Release 条目时退回按 tag 名 `-alpha/-beta/-rc/-dev` 推断）

## 产物命名

```
openssl-<版本>-<编译配置>.tar.gz     例：openssl-3.6.4-ubuntu-22.04-gcc-x86_64.tar.gz
```

## 工作流

```
Detect Versions → Build Matrix (版本 × 配置) → Create Release
```

每个平台执行：`Configure → Build (make/nmake) → Test → Install → Verify → Upload Artifact`

Release 阶段下载同版本所有平台的 Artifacts，打包为 `.tar.gz` 并上传到 GitHub Release。若该版本的 Release 已存在，则**不重建、不覆盖已有资产，只补传缺失的产物**并同步更新说明——因此新增编译平台后重新运行即可把新产物补进历史 Release。

## Release 下载

构建产物发布在 [Releases](https://github.com/zaoanshijie/openssl-build/releases) 页面。

## 手动触发

在 [Actions](https://github.com/zaoanshijie/openssl-build/actions) 页面选择 **Build OpenSSL** → **Run workflow**。

支持可选的 `versions` 参数：填入逗号分隔的版本 tag 可只构建指定版本（例如 `openssl-4.0.2`），留空则按自动检测的最新版本构建。

```bash
gh workflow run build.yml -R zaoanshijie/openssl-build -f versions=openssl-4.0.2
```
