# Qt 编译环境 Docker 镜像

目标：提供可用于编译 Qt 源码的基础环境，使用 `gcc/g++ 9`（`gcc-9`），并安装较新的 CMake（可通过构建参数指定版本），同时支持 `linux/amd64` 与 `linux/arm64`。

## 构建

在 Windows 上构建前，请先启动 Docker Desktop（确保 Linux Engine 可用）。

### 1) 构建当前架构（最简单）

```bash
docker build -f Dockerfile_base -t qt-buildenv:ubuntu20.04-gcc9-cmake .
```

### 2) Multi-arch（amd64 + arm64）

需要 Docker Buildx（以及在非原生架构时的 QEMU 模拟）。

```bash
docker buildx build -f Dockerfile_base \
  --platform linux/amd64,linux/arm64 \
  -t yourrepo/qt-buildenv:ubuntu20.04-gcc9-cmake \
  --push \
  .
```

如果只想本地加载单一平台：

```bash
docker buildx build -f Dockerfile_base \
  --platform linux/amd64 \
  -t qt-buildenv:ubuntu20.04-gcc9-cmake \
  --load \
  .
```

## 可配置项

- `CMAKE_VERSION`：Kitware 预编译包版本（默认 `3.31.6`），例如：

```bash
docker build -f Dockerfile_base \
  --build-arg CMAKE_VERSION=3.31.6 \
  -t qt-buildenv:ubuntu20.04-gcc9-cmake .
```

## 验证

```bash
docker run --rm -it qt-buildenv:ubuntu20.04-gcc9-cmake bash -lc "gcc --version && cmake --version"
```

## Qt 6.8 编译镜像

Dockerfile：`Dockerfile_qt6.8`

示例（先准备 base 镜像 tag 为 `base_version`）：

```bash
BASE_VERSION="$(cat base_version)"
docker build -f Dockerfile_qt6.8 \
  --build-arg BASE_IMAGE=yourname/qt-buildenv \
  --build-arg BASE_VERSION="${BASE_VERSION}" \
  -t yourname/qt6.8:"${BASE_VERSION}" \
  .
```

## GitHub Actions（推送到 Docker Hub）

工作流：`.github/workflows/dockerhub-base.yml`

- base 镜像只在 `base_version` 发生变化时才会构建/推送（因此修改 `Dockerfile_base` 时请同步更新 `base_version`）。
- 镜像仓库名：优先使用 GitHub Variables `DOCKERHUB_REPO`（或 Secrets `DOCKERHUB_REPO`）（例如 `yourname/qt-buildenv`）；若未设置则默认 `${DOCKERHUB_USERNAME}/qt-buildenv`
- 登录凭据：GitHub Secrets `DOCKERHUB_USERNAME` 与 `DOCKERHUB_TOKEN`
- 版本号来源：`base_version`
- 打 tag 规则：若 Docker Hub 上已存在 `${version}` 这个 tag，则本次推送为 `latest`；否则推送为 `${version}`

Qt 6.8 工作流：`.github/workflows/dockerhub-qt68.yml`

- base 仓库：`DOCKERHUB_REPO`（Variables 或 Secrets）
- qt6.8 输出仓库：`DOCKERHUB_QT68_REPO`（Variables 或 Secrets）
