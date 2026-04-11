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
- **启动器版本**（脚本内 `VERSION` 元组）：**2.62**（与默认检出的 `git-repo` 标签 `v2.62` 对齐）
- **默认 `REPO_URL`**：`https://github.com/GerritCodeReview/git-repo.git`（仅访问 GitHub 的服务器可直接 `repo init`）
- **默认 `REPO_REV`**：**`v2.62`**
- **SHA256**（便于校验是否被篡改或下载不完整）：

  ```text
  b36da5e66b3f1ec9fbdbf9ea7a1dd72dd8d2d66b0c8a730dec545d3c59eae07a
  ```

在 **Linux / macOS** 上首次使用前建议赋予执行权限：

```bash
chmod +x bin/repo
```

依赖：**Python 3.6+**、**git**。`repo init` 第一次运行时会 **克隆整套 git-repo 工程**（不仅是启动器）。本仓库已把内置启动器默认改为 **从 GitHub 拉取并固定标签 `v2.62`**；若仍使用未修改的官方启动器，默认会从 `gerrit.googlesource.com` 下载，在无该站点访问权限的环境会卡住。

## 无公网 / 仅内网 Git：自建 `git-repo` 镜像（可选）

若服务器 **不能访问 GitHub**，可将 `git-repo` 推到内网或自建 Git，并通过环境变量或命令行覆盖默认地址，**不必再改脚本**。

本仓库内置启动器当前默认（已改过，与下面片段一致）：

```122:128:bin/repo
REPO_URL = os.environ.get("REPO_URL", None)
if not REPO_URL:
    REPO_URL = "https://github.com/GerritCodeReview/git-repo.git"
REPO_REV = os.environ.get("REPO_REV")
if not REPO_REV:
    REPO_REV = "v2.62"
```

### 步骤 1：在能上网的机器上制作镜像

任选其一作为源：

- 官方：`git clone https://gerrit.googlesource.com/git-repo`
- 或 GitHub 只读镜像：<https://github.com/GerritCodeReview/git-repo>

推送到你的账号（示例仓库名 `git-repo`，请换成你的 URL）：

```bash
cd git-repo
git remote add github git@github.com:luoorshi/git-repo.git   # 先在 GitHub 上建空仓库
git push github --all
git push github --tags
```

务必保证镜像上存在与 **`REPO_REV` 一致的分支或标签**（本仓库默认可用 **`v2.62`**）；若只用 `stable` 或 `main`，请把 `export REPO_REV=...` 改成镜像上真实存在的引用。

### 步骤 2：在无公网服务器上初始化

在每次 `repo init` 前导出变量，或写在 `~/.bashrc` / 工作区 `env.sh` 里 `source`：

```bash
export REPO_URL=git@github.com:luoorshi/git-repo.git
export REPO_REV=v2.62
~/bin/repo init -u git@github.com:luoorshi/repo.git -b main -m default.xml
```

等价写法（仅影响本次命令）：

```bash
REPO_URL=git@github.com:luoorshi/git-repo.git REPO_REV=v2.62 \
  ~/bin/repo init -u git@github.com:luoorshi/repo.git -b main -m default.xml
```

也可使用启动器参数（同样不修改脚本文件）：

```bash
~/bin/repo init -u git@github.com:luoorshi/repo.git -b main -m default.xml \
  --repo-url git@github.com:luoorshi/git-repo.git \
  --repo-rev v2.62
```

说明：服务器只要能 **访问你设的 `REPO_URL`（如 github.com 或公司内网 Git）** 即可；使用本仓库默认配置时 **无需** 访问 `gerrit.googlesource.com`。

### 步骤 3（可选）：改启动器默认地址，免每次 export

本仓库 [`bin/repo`](bin/repo) **已默认** 使用 `https://github.com/GerritCodeReview/git-repo.git` 与 **`v2.62`**。若需改为自有镜像或固定其它标签，可直接编辑其中 **`REPO_URL` / `REPO_REV` 的默认值**（见上文代码片段）。

注意：日后若用官方下载的 `repo` 覆盖 `bin/repo`，需要重新改这两处。

### 步骤 4（可选）：整机离线拷贝 `.repo/repo`

若服务器 **完全不能** `git clone` 任何远程仓库，可在联网机器上在同一版本启动器下执行一次成功的 `repo init`，将整个 **`.repo/repo/`** 目录打成压缩包拷到服务器对应工作区的 `.repo/repo/`，再在同一目录执行后续 `repo` 命令（需与清单 `init` 一致）。此方式维护成本高，仅适合严格隔离环境。

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
