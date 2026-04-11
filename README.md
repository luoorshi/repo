# repo

Linux 源码树多仓库管理（Quark H5 & H3 等）。本仓库为 **manifest 仓库**：仅包含 `default.xml` 与内置的 `repo` 启动脚本，用于 `repo init` / `repo sync` 一次性同步 u-boot、linux、crust、ARM Trusted Firmware 等子仓库，避免编译工具污染各源码树。

## 目录说明

| 路径 | 说明 |
|------|------|
| [default.xml](default.xml) | repo 清单：子项目远程地址与检出路径 |
| [bin/repo](bin/repo) | Google git-repo 官方启动器（随仓库分发，弱网可免单独下载） |

## 内置 `repo` 启动器

- **获取地址**：<https://storage.googleapis.com/git-repo-downloads/repo>
- **本仓库内文件**：`bin/repo`
- **启动器版本**（脚本内 `VERSION` 元组）：**2.54**
- **SHA256**（便于校验是否被篡改或下载不完整）：

  ```text
  11bc6893e9e0c0940fc1cc95b75c645f9a29fca879d89ceaa898a4d761a2add7
  ```

在 **Linux / macOS** 上首次使用前建议赋予执行权限：

```bash
chmod +x bin/repo
```

依赖：**Python 3.6+**、**git**。完整 `repo sync` 仍可能通过网络拉取 `repo` 工具本体（默认 `REPO_REV=stable`）；若完全离线，需在能联网环境先完成一次 `repo init` 与 `repo sync`，或将 `.repo/repo` 一并打包迁移。

## 使用方法（Linux 上推荐）

将本仓库作为工作区根目录，或先克隆再进入目录：

```bash
git clone <你的_manifest_仓库_URL> my-workspace
cd my-workspace
export PATH="$PWD/bin:$PATH"

# -u：本 manifest 仓库的 git URL
# -b：manifest 所在分支（例如 main 或 master）
repo init -u <你的_manifest_仓库_URL> -b <分支名> -m default.xml
repo sync -j"$(nproc)"
```

也可不显式指定 `-m default.xml`（repo 默认即读取 `default.xml`）。

同步完成后，各子仓库相对路径为：

| 目录 | 内容 |
|------|------|
| `u-boot/` | `git@github.com:luoorshi/u-boot.git` |
| `linux/` | `git@github.com:luoorshi/linux.git` |
| `crust/` | <https://github.com/crust-firmware/crust> |
| `arm-trusted-firmware/` | <https://github.com/TrustedFirmware-A/trusted-firmware-a.git>（分支 **main**） |

## 分支说明

- 清单中 `<default revision="master"/>` 适用于未单独写 `revision` 的项目。若你的 `luoorshi/u-boot`、`luoorshi/linux` 默认分支为 **main**，请在 [default.xml](default.xml) 里给对应 `<project>` 增加 `revision="main"`，或把 `<default revision="..."/>` 改为 **main**。
- `TrustedFirmware-A/trusted-firmware-a` 已固定为 **`revision="main"`**（与 GitHub 默认一致）。

## 预留槽位（尚未启用）

以下项目在 [default.xml](default.xml) 中仅以 **XML 注释** 保留模板，**未**加入有效 `<project>`，避免 `repo sync` 因仓库不存在而失败：

- **tools**：工具链 / 预编译工具等（待建仓库后取消注释并改 `name` / `revision`）。
- **build-scripts**：编译、打包脚本（同上）。
- **ubuntu-rootfs**：Ubuntu 根文件系统通常体积大，不一定适合整树进 Git；建议约定工作区下目录名 `ubuntu-rootfs/`，通过 rsync、内网镜像或解压 rootfs 镜像填充；若日后有专用仓库，再按注释模板增加 `<project>`。

## 全志 H5 与 Crust / ATF

深睡、ATF 与 Crust 的打包、or1k 交叉编译链等仍遵循上游文档，例如 [crust-firmware/crust](https://github.com/crust-firmware/crust) 的 README。本清单只负责把各 **Git 仓库** 固定到上述相对路径，不替代板级构建流程。

## Windows 说明

`repo` 主要在 Linux/macOS 使用；若在 Windows 上编辑清单，同步与编译仍建议在 Linux 开发机上执行 `repo init` / `repo sync` 做验证。
