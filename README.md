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
  <p>
    <a href="README.zh.md">
      <img src="https://img.shields.io/badge/README-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-brightgreen" alt="简体中文" />
    </a>
  </p>
</div>

Automated builder for the latest stable Slurm Debian packages (.deb). The workflow runs on every push and daily at 10:00 UTC+8, uploads build artifacts, and creates a `vX.Y.Z` tag and GitHub Release.

### Features
- **Auto-detect latest version**: Parse the newest `slurm-X.Y.Z.tar.*` from SchedMD's download index.
- **Debian packaging**: Use upstream Debian packaging with `dpkg-buildpackage` to produce multiple binary packages.
- **Requested components enabled via build deps**:
  - **acct_gather_energy/ipmi** (`freeipmi`)
  - **acct_gather_interconnect/ofed** (`libibmad`, `libibumad`, etc.)
  - **acct_gather_profile/hdf5** (`libhdf5-dev`)
  - **accounting_storage/mysql** (`default-libmysqlclient-dev`)
  - **auth/munge** (`libmunge-dev`)
  - **auth/slurm (jwt)** (`libjwt-dev`; inject `--with-jwt` into `debian/rules` if needed)
  - **AutoDetect=nvml** (`libnvidia-ml-dev` or CUDA toolkit as fallback)
  - **PAM support** (`libpam0g-dev`)
  - **task/affinity** (`libnuma-dev`)
  - **task/cgroup** (`libhwloc-dev`, `libbpf-dev`, `libdbus-1-dev`)
  - **slurmrestd** (`libjson-c-dev`, `libhttp-parser-dev`, `libyaml-dev`, `libjwt-dev`)
  - **HTML man pages** (`man2html`)
  - Plus Readline, PMIx, GTK2 (sview) where applicable
- **Artifacts and Release**: Upload `.deb` and `*.changes` as artifacts and create a `vX.Y.Z` release.

Reference: [Quick Start Administrator Guide](https://slurm.schedmd.com/quickstart_admin.html)

### Triggers
- **Push** to any branch
- **Schedule**: daily at 10:00 UTC+8 (`0 2 * * *`)
- **Manual**: Workflow Dispatch

Workflow: `/.github/workflows/main.yaml`.

### Output Packages
Typically (actual results may vary):
- `slurmctld_*.deb`
- `slurmdbd_*.deb`
- `slurmd_*.deb`
- `slurmrestd_*.deb`
- `slurm-client_*.deb`
- `slurm-libpmi*_*.deb`
- `slurm-dev_*.deb`
- plus meta `slurm-<version>_*.deb` and `*.changes`

### Get the build results
- After a PR or push run, download from the run's **Artifacts**.
- Each successful build publishes a `vX.Y.Z` **Release** with all `.deb` files.

### Install example
```bash
sudo apt-get update
sudo apt-get install -y ./slurmctld_*.deb ./slurmd_*.deb ./slurm-client_*.deb
# For RESTD, DBD, dev packages, or PMI, install additional .deb as needed
```
> Note: `accounting_storage/mysql` means the plugin is built. You still need to deploy and configure the database and `slurmdbd`.

### Customize
- Pin a specific Slurm version by modifying the "discover latest version" step in `main.yaml`.
- Change the schedule via `on.schedule.cron` (default `0 2 * * *`).
- Adjust build dependencies or injected `configure` flags to match your needs.

### Develop & debug
- Manually run via GitHub Actions → Run workflow.
- If features are missing, ensure the proper `*-dev` packages are installed and check whether `debian/rules` needs `--with-*`/`--enable-*`.

### Star History
[![Star History Chart](https://api.star-history.com/svg?repos=jingyijun/slurm-builder&type=Date)](https://star-history.com/#jingyijun/slurm-builder&Date)

### License
- MIT License. See `LICENSE`.
