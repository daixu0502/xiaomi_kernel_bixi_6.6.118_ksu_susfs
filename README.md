# Xiaomi bixi Android 15 GKI Kernel

面向 Xiaomi `bixi` 的 Android 15 GKI 6.6 自定义内核自动构建项目，集成官方 KernelSU 与 SUSFS，并通过 GitHub Actions 自动生成 `boot.img`、SUSFS 模块和 GitHub Release。

## 当前构建配置

| 项目 | 版本/配置 |
|---|---|
| 设备代号 | Xiaomi `bixi` |
| Android | Android 15 |
| GKI | 6.6 |
| Linux | 6.6.118 |
| 页面大小 | 4K |
| Manifest | `common-android15-6.6-2026-01` |
| 内核标签 | `android15-6.6-2026-01_r37` |
| 内核提交 | `c4127a25dcf3` |
| BUILD_NUMBER | `15863337` |
| KernelSU | 官方 KernelSU |
| KernelSU 固定提交 | `932014ab5b2c9b74a3d11e2ec4d17dd10fc9442e` |
| KernelSU 内核版本 | `32601` |
| SUSFS | v2.3.0 |
| SUSFS 固定提交 | `eba2a88a5ba303e3d79d08e0717b956e9cf784a1` |
| 内核后缀 | `-Jaco-KSU-SUSFS` |
| Boot Header | v4 |
| Boot Ramdisk | 空 |
| Boot 分区大小 | 100663296 字节（96 MiB） |
| AVB Footer | SHA-256、`algorithm NONE` |

生成的内核版本类似：

```text
6.6.118-android15-8-gc4127a25dcf3-Jaco-KSU-SUSFS-ab15863337-4k
```

## 功能

- Android 15 GKI 6.6 内核
- 官方 KernelSU
- SUSFS v2.3.0
- 4K 页面大小
- 保留 `CONFIG_MODVERSIONS=y`
- 关闭 `CONFIG_MODULE_SIG_PROTECT`
- 关闭强制模块签名校验
- 兼容原厂 Wi-Fi、蓝牙等厂商模块
- 自动生成 header v4 `boot.img`
- 自动构建 SUSFS KernelSU 模块
- 自动生成 SHA-256 校验文件
- 自动上传 GitHub Actions Artifact
- 自动创建或更新 GitHub Release

## GitHub Actions 构建

工作流文件位于：

```text
.github/workflows/build-kernel.yml
```

进入 GitHub 仓库：

```text
Actions → Build bixi GKI KernelSU SUSFS → Run workflow
```

需要填写以下参数：

| 参数 | 说明 | 默认值 |
|---|---|---|
| `release_tag` | GitHub Release 标签 | 手动填写 |
| `build_number` | Android 内核构建编号 | `15863337` |
| `kernelsu_ref` | KernelSU 标签或提交 | `932014ab...` |
| `susfs_ref` | SUSFS 兼容提交 | `eba2a88a...` |

Release 标签示例：

```text
v6.6.118-jaco-1
```

首次运行前，请在 GitHub 仓库中打开：

```text
Settings → Actions → General → Workflow permissions
```

选择：

```text
Read and write permissions
```

否则工作流可能无法创建 GitHub Release。

## 构建产物

构建成功后，GitHub Release 和 Actions Artifact 中会包含：

```text
boot-bixi-6.6.118-Jaco-KSU-SUSFS.img
ksu_module_susfs.zip
SHA256SUMS.txt
BUILD-INFO.txt
```

各文件用途：

- `boot-*.img`：可通过 Fastboot 测试或刷入的启动镜像。
- `ksu_module_susfs.zip`：通过 KernelSU 管理器安装的 SUSFS 用户空间模块。
- `SHA256SUMS.txt`：构建产物的 SHA-256 校验值。
- `BUILD-INFO.txt`：内核、KernelSU、SUSFS 提交和版本信息。

## 刷入前提

必须满足：

- Bootloader 已解锁。
- 设备代号为 `bixi`。
- ROM 使用兼容的 Android 15 GKI 6.6 内核。
- Boot 镜像使用 header v4。
- Boot ramdisk 为空。
- Boot 分区大小为 96 MiB。
- 已备份当前槽位的原厂 `boot.img`。
- 已准备可用的 Fastboot 环境。

本项目生成的镜像使用：

```text
AVB Algorithm: NONE
```

它没有小米原厂 RSA4096 签名，不能在 Bootloader 锁定状态下启动。

## Windows 下测试启动

进入 Platform Tools：

```bat
cd /d D:\XiaoMI-Phone\platform-tools
```

进入 Bootloader：

```bat
adb reboot bootloader
```

查询当前槽位：

```bat
fastboot getvar current-slot
```

如果设备支持临时启动，建议先执行：

```bat
fastboot boot boot-bixi-6.6.118-Jaco-KSU-SUSFS.img
```

确认系统、Wi-Fi、蓝牙、KernelSU 和 SUSFS 均正常后，再刷入当前槽位。

当前槽位为 `a`：

```bat
fastboot flash boot_a boot-bixi-6.6.118-Jaco-KSU-SUSFS.img
```

当前槽位为 `b`：

```bat
fastboot flash boot_b boot-bixi-6.6.118-Jaco-KSU-SUSFS.img
```

重启：

```bat
fastboot reboot
```

不要在未确认当前槽位的情况下直接复制刷写命令。

## 安装 SUSFS 模块

系统正常启动后：

1. 打开 KernelSU 管理器。
2. 进入“模块”。
3. 选择“从本地安装”。
4. 选择 `ksu_module_susfs.zip`。
5. 安装完成后重启设备。

官方 SUSFS 模块不包含 WebUI，这是正常现象。命令行工具位于：

```text
/data/adb/ksu/bin/ksu_susfs
```

## 验证内核

Windows 下执行：

```bat
adb shell uname -r
```

预期输出类似：

```text
6.6.118-android15-8-gc4127a25dcf3-Jaco-KSU-SUSFS-ab15863337-4k
```

检查 Root：

```bat
adb shell
su
id
```

预期包含：

```text
uid=0(root)
```

检查 SUSFS：

```sh
/data/adb/ksu/bin/ksu_susfs show version
/data/adb/ksu/bin/ksu_susfs show enabled_features
/data/adb/ksu/bin/ksu_susfs show variant
```

检查运行日志：

```sh
dmesg | grep -i susfs
```

## 模块兼容性

为了兼容原厂 Wi-Fi、蓝牙及其他厂商模块，本项目保留：

```text
CONFIG_MODVERSIONS=y
```

并确保以下选项未启用：

```text
CONFIG_MODULE_SIG_PROTECT
CONFIG_MODULE_SIG_FORCE
```

如果刷入后 Wi-Fi 或蓝牙无法开启，请检查：

- 内核源码提交是否与原厂内核匹配。
- `uname -r` 是否保持相同的 GKI/KMI 基线。
- 是否意外开启模块签名保护。
- `CONFIG_MODVERSIONS` 是否仍为 `y`。
- ROM 是否已升级到不同的厂商模块版本。

## KernelSU 版本显示异常

如果 KernelSU 管理器显示类似：

```text
16-2
16-4
```

通常是 Bazel 沙箱无法读取 KernelSU 的 Git 历史，触发了兜底版本。

构建时必须使用：

```text
--strategy=KernelBuild=local
```

同时 KernelSU 仓库必须包含完整 Git 历史。正常情况下，本项目固定提交对应的内核版本为：

```text
32601
```

## 更新 KernelSU 或 SUSFS

不要直接同时更新到两个项目的最新 `main`。

推荐流程：

1. 固定新的 KernelSU 标签或提交。
2. 获取 `gki-android15-6.6` 分支最新 SUSFS 提交。
3. 使用 `git apply --check` 检查两个补丁。
4. 完成一次完整内核编译。
5. 真机测试开机、Wi-Fi、蓝牙、KernelSU 和 SUSFS。
6. 测试全部通过后记录 SUSFS 提交哈希。
7. 更新工作流中的默认提交。

SUSFS 提交无法根据 KernelSU 版本号直接计算，必须通过补丁、编译和真机测试确定。

## 兼容范围

虽然 AVB Footer 中没有写入 ROM 指纹、安全补丁日期和系统版本，但这不代表镜像能跨设备或跨任意 ROM 通用。

本项目只面向：

```text
Xiaomi bixi
Android 15
GKI 6.6
4K pages
兼容的厂商模块/KMI
```

请勿刷入其他设备、其他页面大小或不兼容的内核基线。

## 上游项目

- [Android Kernel Manifest](https://android.googlesource.com/kernel/manifest)
- [KernelSU](https://github.com/tiann/KernelSU)
- [susfs4ksu](https://gitlab.com/simonpunk/susfs4ksu)

## 免责声明

解锁 Bootloader、修改内核和刷写启动镜像存在无法启动、数据丢失或设备损坏风险。

使用者应自行备份原厂镜像和重要数据，并确认具备恢复 Fastboot 镜像的能力。本项目不提供任何设备安全、数据完整性或第三方应用兼容性保证。