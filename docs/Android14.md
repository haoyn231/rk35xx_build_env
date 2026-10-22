# RK3576 Android 14 编译环境

此环境用于 RK3576 Android 14 SDK，基于 Ubuntu 22.04。它独立存放于 `docker/android14/`，可与原有 Linux SDK 环境并存。

## 已验证范围

原始环境已使用 `rk3576-android14.0-SDK-20260616` 完成：

- `pro-rk3576-mid` 的 U-Boot、Linux 6.1、Android 14 和 `update.img` 构建。
- DshanPi A1 适配分支的 U-Boot、Linux 6.1、Android 14 和 `update.img` 构建。

以上是编译和打包验证，尚未上板验证。DshanPi A1 的板型文件来自 SDK 的独立适配分支；容器本身不会为其他 SDK 添加该板型。

## 主机要求

- Linux x86_64 主机，安装 Docker Engine 和 Docker Compose v2。
- SDK 应包含 Linux x86 的预编译工具链，以及 `prebuilts/jdk/jdk8/linux-x86` 和 Android 构建所用 JDK。
- 为源码和构建输出准备充足磁盘空间，建议解压源码后仍有至少 200 GiB 可用空间。
- 本机约 30 GiB 内存加 7.5 GiB zram 曾在 Soong 阶段发生 OOM；增加 32 GiB 磁盘 swap 后，以 `-J8`、`NINJA_HIGHMEM_NUM_JOBS=1` 完成构建。其他 SDK 的内存需求可能不同。

## 构建容器镜像

从本仓库根目录执行，将 `SDK_PATH` 设置为已解压 SDK 根目录的绝对路径：

```bash
export SDK_PATH=/absolute/path/to/rk3576-android14.0-SDK-20260616
export HOST_UID=$(id -u)
export HOST_GID=$(id -g)

docker compose -f docker/android14/compose.yaml config --quiet
docker compose -f docker/android14/compose.yaml build
docker compose -f docker/android14/compose.yaml up -d
docker compose -f docker/android14/compose.yaml exec rk3576-android-builder bash
```

SDK 挂载至容器的 `/workspace`，ccache 使用独立 Docker 卷保存。Compose 会检查 SDK 路径是否存在，避免路径写错时自动创建空目录。UID/GID 构建参数使容器生成的文件归宿主用户所有。

容器名为 `rk3576-android14-build-env`，镜像名为 `rk3576-android14-build-env:local`。如本机已有同名容器，请使用已有容器或先调整 Compose 中的名称。

## 编译 SDK

以下命令在容器的 `/workspace` 执行，适用于已验证 SDK 自带的 Rockchip `build.sh`：

```bash
# 首次构建选择目标板型
./build.sh init

# 编译 U-Boot、内核、Android，并打包和归档固件
NINJA_HIGHMEM_NUM_JOBS=1 ./build.sh -UKAup -J8
```

选择板型时核对菜单中的名称，菜单编号会随板型数量变化。其他厂商 SDK 应以各自文档中的初始化和编译入口为准。

常用选项：

| 选项 | 功能 |
| --- | --- |
| `-U` | 编译 U-Boot |
| `-K` | 编译内核及相关驱动 |
| `-A` | 编译 Android 系统 |
| `-u` | 生成 `update.img` |
| `-p` | 归档至 `IMAGE/` |
| `-J8` | 设置 8 路编译 |

单独使用 `-J8` 不会选择完整构建阶段。仅使用 `-UK` 时，脚本会重打包 boot，但不会重编 Android 系统；修改 Android 产品配置后应执行 `-A`。

固件输出位置：

```text
rockdev/Image-rk3576_u/               # 分区镜像和 update 包
IMAGE/<构建版本>/IMAGES/             # 归档镜像
```

## 兼容性说明

- 从源码安装 Python 2.7.18，供旧版 U-Boot 工具使用。`/opt/python2/bin` 在 PATH 前部，因此 `python` 和 `python2` 指向 Python 2.7.18；Python 3 使用 `python3` 调用。
- `/usr/lib/jvm/java-8-openjdk-amd64` 指向 SDK 内置 JDK 8，以满足厂商检查脚本；Android 14 构建使用 SDK 内置 JDK 17。
- 安装 32 位兼容库和 64 位 `libncurses5`，满足旧版工具链与 RenderScript Clang 的运行依赖。
- 包含 `file` 命令，用于 U-Boot 充电资源打包脚本。
- 已验证板型的 `uboot.img` 为包含 Trust 的 FIT 镜像，脚本提示没有单独的 `trust.img` 时应结合实际打包结果判断。
- Compose 沿用当前构建环境的特权模式与主机网络配置，启动后容器会获得这些主机权限。

## 停止容器

在宿主机的本仓库根目录执行，保留前文导出的环境变量：

```bash
docker compose -f docker/android14/compose.yaml down
```

该命令保留 SDK 文件和 ccache 卷。
