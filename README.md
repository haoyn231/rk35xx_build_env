# rk35xx SDK 编译环境
基于 Ubuntu22.04 基础镜像，提供 Dockerfile 和 Docker Compose 文件。

## 环境选择

| 环境 | 配置文件 | 使用说明 |
| --- | --- | --- |
| 原有 RK35xx Linux SDK 环境 | `docker/docker-compose.yml` | 见下文快速开始 |
| RK3576 Android 14 SDK 环境 | `docker/android14/compose.yaml` | [Android 14 编译指南](./docs/Android14.md) |

Android 14 环境基于本地已完成 U-Boot、Linux 6.1、Android 系统及 `update.img` 构建的配置独立添加，包含 Python 2.7.18、32 位兼容库和 SDK JDK 路径配置。SDK 源码需自行准备；本仓库不包含 Android SDK 或编译固件。

适用于原厂 SDK 或者各个开发板厂家二次开发的 SDK，目前测试可用的有：

rk3506 (kernel: rockchip-6.1)
- 万象奥科 rk3506 系列
- 触觉智能 rk3506 系列
- Luckfox Lyra 系列

rk3576 (kernel: rockchip-6.1)
- 100ask Dshanpi-A1
- [Luckfox Omni 系列](./docs/Luckfox%20Omni.md)

rv1126b (kernel: rockchip-6.1)
- Luckfox Aura 系列

> 欢迎进行测试和补充～

## 快速开始
容器内目录说明：
```bash
/workspace
├── 100ask-rk3576_SDK/       #<--- sdk本体和压缩包
├── 100ask-rk3576_SDK.tar.gz #<--- 需要被gitignore掉
├── docker/
│   ├── docker-compose.yml
│   └── Dockerfile
├── LICENSE
└── README.md
```


启动docker
```bash
# 进入存放Dockerfile的目录
cd docker

# 创建容器
docker compose up -d

# 进入
docker exec -it rk35xx_build_env bash
```
