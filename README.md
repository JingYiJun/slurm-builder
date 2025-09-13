<div align="center">
  <h1>slurm-builder</h1>
  <p>
    <a href="https://github.com/jingyijun/slurm-builder/actions/workflows/main.yaml">
      <img src="https://github.com/jingyijun/slurm-builder/actions/workflows/main.yaml/badge.svg" alt="CI" />
    </a>
    <a href="https://github.com/jingyijun/slurm-builder/releases">
      <img src="https://img.shields.io/github/v/release/jingyijun/slurm-builder?sort=semver" alt="Release" />
    </a>
    <a href="LICENSE">
      <img src="https://img.shields.io/github/license/jingyijun/slurm-builder" alt="License" />
    </a>
    <a href="https://github.com/jingyijun/slurm-builder/stargazers">
      <img src="https://img.shields.io/github/stars/jingyijun/slurm-builder?style=social" alt="Stars" />
    </a>
    <img src="https://img.shields.io/github/last-commit/jingyijun/slurm-builder" alt="Last Commit" />
  </p>
</div>

自动化构建 Slurm 最新稳定版的 Debian 包（.deb），并在 Push 或每天 UTC+8 10:00 触发。构建完成后会上传构建产物为 GitHub Actions Artifacts，同时以 `vX.Y.Z` 形式打 tag 并创建 Release。

### 功能特点
- **自动发现最新版本**：从 SchedMD 官方下载目录解析最新稳定版 `slurm-X.Y.Z.tar.*`。
- **按 Debian 规范打包**：使用上游自带的 Debian 打包规则，`dpkg-buildpackage` 产出多个二进制包。
- **启用所需组件（通过依赖编译）**：
  - **acct_gather_energy/ipmi**（`freeipmi`）
  - **acct_gather_interconnect/ofed**（`libibmad`、`libibumad` 等）
  - **acct_gather_profile/hdf5**（`libhdf5-dev`）
  - **accounting_storage/mysql**（`default-libmysqlclient-dev`）
  - **auth/munge**（`libmunge-dev`）
  - **auth/slurm (jwt)**（`libjwt-dev`；在需要时对 `debian/rules` 注入 `--with-jwt`）
  - **AutoDetect=nvml**（`libnvidia-ml-dev` 或 CUDA 工具包兜底）
  - **PAM support**（`libpam0g-dev`）
  - **task/affinity**（`libnuma-dev`）
  - **task/cgroup**（`libhwloc-dev`、`libbpf-dev`、`libdbus-1-dev`）
  - **slurmrestd**（`libjson-c-dev`、`libhttp-parser-dev`、`libyaml-dev`、`libjwt-dev`）
  - **HTML man pages**（`man2html`）
  - 另外启用 **Readline**、**PMIx**、**GTK2**（sview）等能力的依赖安装
- **产物上传与版本发布**：将 `.deb` 及相关 `*.changes` 上传为 Artifact，并创建 `vX.Y.Z` Release。

参考： [Quick Start Administrator Guide](https://slurm.schedmd.com/quickstart_admin.html)

### 工作流触发
- **Push** 到任意分支
- **定时**：每天 UTC+8 10:00（即 `0 2 * * *`）
- **手动**：Workflow Dispatch

工作流文件位于 `/.github/workflows/main.yaml`。

### 构建产物
典型会生成（以实际打包结果为准）：
- `slurmctld_*.deb`
- `slurmdbd_*.deb`
- `slurmd_*.deb`
- `slurmrestd_*.deb`
- `slurm-client_*.deb`
- `slurm-libpmi*_*.deb`
- `slurm-dev_*.deb`
- 以及 `slurm-<version>_*.deb` 汇总包与 `*.changes`

### 如何获取构建结果
- 打开的 Pull Request 或 Push 构建完成后，可在该运行记录的 **Artifacts** 下载。
- 每次成功构建还会以 `vX.Y.Z` 创建 **Release** 并上传所有 `.deb`。

### 安装示例
```bash
sudo apt-get update
sudo apt-get install -y ./slurmctld_*.deb ./slurmd_*.deb ./slurm-client_*.deb
# 如需 RESTD、DBD、开发包或 PMI，请额外安装对应 .deb
```
> 注意：`accounting_storage/mysql` 仅代表编译出 MySQL/MariaDB 存储插件。实际运行需自行部署并配置数据库与 `slurmdbd`。

### 自定义
- 如需固定构建某个 Slurm 版本，可修改 `main.yaml` 中“发现最新版本”的步骤，指定下载的 tarball。
- 如需调整定时构建时间，修改 `on.schedule.cron` 表达式（当前为每天 `0 2 * * *`）。
- 可根据需要在“Install build dependencies”步骤中增删依赖，或在“Prepare Debian build”中调整 `configure` 参数注入逻辑。

### 开发与调试
- 手动执行一次构建：在 GitHub Actions 页面选择该 Workflow，使用 **Run workflow**。
- 若构建缺少某些特性，优先检查对应开发库（`*-dev`）是否已安装；其次查看 `debian/rules` 是否需要添加 `--with-*`/`--enable-*`。

### Star History
[![Star History Chart](https://api.star-history.com/svg?repos=jingyijun/slurm-builder&type=Date)](https://star-history.com/#jingyijun/slurm-builder&Date)

### 许可证
- 本仓库使用 **MIT License**，详见 `LICENSE`。
