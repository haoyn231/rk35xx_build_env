# RK3576 Android 14 Docker 环境

基于 Ubuntu 22.04，配置文件位于 `docker/android14/`。

## 环境依赖

- Linux x86_64 主机，安装 Docker Engine 和 Docker Compose v2。
- 自行准备并解压 RK3576 Android 14 SDK。镜像使用 SDK 内的工具链与 JDK，其中 JDK 8 路径为 `prebuilts/jdk/jdk8/linux-x86`。

镜像包含 Python 2.7.18、Python 3、32 位兼容库、ccache 及常用编译工具。`python` 和 `python2` 指向 Python 2.7.18，Python 3 使用 `python3` 调用。

## 启动

在本仓库根目录执行：

```bash
export SDK_PATH=/absolute/path/to/android14-sdk
export HOST_UID=$(id -u)
export HOST_GID=$(id -g)

docker compose -f docker/android14/compose.yaml build
docker compose -f docker/android14/compose.yaml up -d
docker compose -f docker/android14/compose.yaml exec rk3576-android-builder bash
```

进入容器后，在 `/workspace` 按 SDK 自带文档执行编译命令。

## 配置说明

| 配置 | 说明 |
| --- | --- |
| `SDK_PATH` | 必填，已解压 SDK 的绝对路径，挂载至 `/workspace`；目录必须存在 |
| `HOST_UID` / `HOST_GID` | 构建镜像时设置 builder 用户的 UID/GID，默认均为 `1000`，用于匹配宿主文件权限 |
| ccache | 使用独立 Docker 卷，容器内目录为 `/home/builder/.ccache` |
| 容器名称 | `rk3576-android14-build-env` |
| 镜像名称 | `rk3576-android14-build-env:local` |

Compose 启用特权模式和主机网络。JDK 8 的兼容路径 `/usr/lib/jvm/java-8-openjdk-amd64` 指向挂载 SDK 中的 `prebuilts/jdk/jdk8/linux-x86`。

## 停止

在宿主机的本仓库根目录执行，保留上述环境变量：

```bash
docker compose -f docker/android14/compose.yaml down
```

SDK 文件和 ccache 卷会保留。
