# lgcorzo/pkger

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Dark Gravity](https://img.shields.io/badge/Autonomous%20Factory-Dark%20Gravity-blueviolet?style=flat-square)](https://github.com/lgcorzo/rust_CACD_autonomous_factory)

> [!NOTE]
> **UPSTREAM STATUS & SOVEREIGN FORK PURPOSE:**
> Following upstream repository archiving and licensing shifts across the MinIO ecosystem, this repository is **actively maintained and hardened** under **@lgcorzo** as an essential infrastructure component of the **[Dark Gravity Autonomous CA/CD Factory](https://github.com/lgcorzo/rust_CACD_autonomous_factory)** ([Architecture Knowledge Base](https://notebook.google.com/notebook/116d5f03-2fb0-4fa7-bd7c-1c9849920abc)). Continuous security auditing, zero-CVE verification, and hardware SIMD optimizations are independently maintained to guarantee supply-chain sovereignty and operational continuity.

---

## 🛡️ Dark Gravity Factory: Technological Sovereignty & Security Rationale

### 1. Architectural Role in Dark Gravity
In the **Dark Gravity Autonomous CA/CD Software Factory (V7.2 / V7.3)**, Packaging toolchain generating reproducible Debian, RPM, and APK packages for sovereign storage binaries deployed on Dark Gravity nodes.

### 2. Supply-Chain Sovereignty & Independent Maintenance
- **Insulation from Upstream License Closures:** When upstream open-source projects close distributions or alter licensing terms, Dark Gravity enforces **semantic mirroring and sovereign forks** to guarantee total independence from commercial vendors.
- **Autonomous Security Verification & Zero-CVE Policy:** This repository is continuously audited for CVEs and vulnerabilities, maintaining a 100% clean secret scanning posture. Regressions and upstream bugs are resolved directly within the autonomous factory ecosystem.
- **Compliance & Provenance:** Ensures cryptographic integrity and auditable lineage in compliance with the **EU AI Act**, **SOC 2 Type II**, and **Hazitek / SPRI** regulatory frameworks.

---

## pkger

pkger is a packaging tool for MinIO projects that generates DEB, RPM, and APK packages along with download metadata JSON files consumed by min.io/download.

## Supported apps

Every app packages for `linux/amd64` and `linux/arm64` only.

| `--appName` | Release dir        | Binary read | Package name | Versioning |
| ----------- | ------------------ | ----------- | ------------ | ---------- |
| `aistor`    | `minio-release`    | `minio`     | `minio`      | date-based |
| `ac`        | `mc-release`       | `mc`        | `mcli`       | date-based |
| `sidekick`  | `sidekick-release` | `sidekick`  | `sidekick`   | date-based |
| `warp`      | `warp-release`     | `warp`      | `warp`       | semver     |
| `memkv`     | `memkv-release`    | `memkv`     | `memkv`      | date-based |
| `aimem`     | `aimem-release`    | `aimem`     | `aimem`      | date-based |
| `minfs`     | `minfs-release`    | `minfs`     | `minfs`      | date-based |

`aistor` and `ac` keep the historical on-disk layout (`minio-release/`, `mc-release/`, `minio`/`mc` binaries, `minio`/`mcli` packages) — only the app name is rebranded. Override any of it with `--releaseDir`, `--binary-name` and `--package-name`.

## Packaging aistor during development

For testing aistor packages during development, first install pkger so it's available in your PATH. Then prepare a release directory (such as `dist`) with architecture-specific subdirectories. For example, create `./dist/linux-amd64` and move your compiled binary there, renaming it to include the release version like `minio.RELEASE.2025-03-12T00-00-00Z.debug.GIT_TAG`. Make sure to replace the timestamp and git tag with your actual values.

You'll also need the minio.service systemd file, which you can download from the minio-service repository:

```shell
wget -O minio.service "https://raw.githubusercontent.com/minio/minio-service/refs/heads/master/linux-systemd/minio.service"
```

Then run pkger with the release version, specifying aistor as the app name and using the `--ignore` flag to continue even if some architectures are missing:

```shell
pkger -r RELEASE.2025-03-12T00-00-00Z.debug.GIT_TAG --appName aistor --ignore --releaseDir=dist
```

The packaged files (rpm, deb, apk) along with `downloads-aistor.json` will be generated in the `./dist` directory.

## Packaging sidekick releases

Sidekick releases follow a similar workflow. Create the release directory structure with a subdirectory per architecture, then place your compiled sidekick binaries in them with the release version appended to the filename.

```shell
mkdir -p ./sidekick-release/linux-amd64 ./sidekick-release/linux-arm64
mv ./sidekick-linux-amd64 ./sidekick-release/linux-amd64/sidekick.RELEASE.2025-03-12T00-00-00Z
mv ./sidekick-linux-arm64 ./sidekick-release/linux-arm64/sidekick.RELEASE.2025-03-12T00-00-00Z
```

Run pkger with the sidekick app name. By default, it builds all three package formats (RPM, DEB, APK), though you can limit this with the `--packager` flag:

```shell
pkger -r RELEASE.2025-03-12T00-00-00Z --appName sidekick
```

The generated packages will appear in the architecture-specific directories along with `downloads-sidekick.json`, which contains download URLs and installation instructions. Note that while APK packages are built, only RPM and DEB installation instructions are included in the JSON metadata.

## Packaging warp releases

Warp uses semantic versioning (e.g., v0.4.3) instead of date-based release tags. The version must include the `v` prefix when you run pkger, but this prefix is automatically stripped in the generated package filenames to follow standard RPM and DEB naming conventions.

Set up the release directories for amd64 and arm64:

```shell
mkdir -p ./warp-release/linux-amd64 ./warp-release/linux-arm64
mv ./warp-linux-amd64 ./warp-release/linux-amd64/warp.v0.4.3
mv ./warp-linux-arm64 ./warp-release/linux-arm64/warp.v0.4.3
```

Run pkger with the semantic version. The version must start with `v` and follow the X.Y.Z format:

```shell
pkger -r v0.4.3 --appName warp
```

The output includes Linux packages (RPM, DEB, APK) in the architecture directories, along with `downloads-warp.json`. This JSON file includes cross-platform download information for Linux (binary, RPM, DEB), macOS (arm64 binary only), and Windows (amd64 binary). Like sidekick, APK packages are built for Linux but only RPM and DEB installation instructions are documented in the JSON.

## Renaming a binary / package (`--binary-name`, `--package-name`)

Two optional flags let you rename what a package installs without breaking existing deployments. Both default to the per-app convention, so omitting them produces exactly the same output as before.

- `--binary-name` overrides the source binary base name read from the release directory (`<releaseDir>/<os>-<arch>/<binary-name>.<release>`) and the raw-binary filename used in the downloads metadata.
- `--package-name` overrides the package name and the installed command under `/usr/local/bin`.

When `--package-name` differs from the app's default package name, that old name is treated as a legacy name: the package installs a back-compat symlink `/usr/local/bin/<old> -> <new>` and declares `provides`/`replaces`/`conflicts` on the old name so the previous package is superseded on install.

For example, shipping the aistor server's packages as `aistor` while leaving the built binary — and the `minio` command customers already invoke — alone:

```shell
pkger -r RELEASE.2025-03-12T00-00-00Z --appName aistor --package-name aistor
```

and the client, whose packages become `acli` while the installed command stays `mcli`:

```shell
pkger -r RELEASE.2025-03-12T00-00-00Z --appName ac --package-name acli
```

Note that neither example passes `--binary-name`: the binary read out of the release directory stays `minio`/`mc`, so no new binary has to be built for the rename.

### Existing download links keep working

A rename changes the package filename, which would break every already-published URL built from the old name. pkger therefore symlinks the old names onto the new package, so both resolve:

```text
aistor-<version>-1.x86_64.rpm             # the real package
aistor.rpm                             -> aistor-<version>-1.x86_64.rpm
minio.rpm                              -> aistor-<version>-1.x86_64.rpm
minio-<version>-1.x86_64.rpm           -> aistor-<version>-1.x86_64.rpm
minio-<version>-1.x86_64.rpm.sha256sum -> aistor-<version>-1.x86_64.rpm.sha256sum
```

The same applies to DEB and APK. The downloads metadata JSON points at the new (real) filenames; the old ones remain reachable as symlinks. Without `--package-name` no legacy links are emitted, since there is nothing to alias.

### Upgrading across the rename

On DEB and RPM the renamed package supersedes the old one automatically via the standard install commands — `dpkg -i` handles the `Replaces`+`Conflicts` takeover and `dnf`/`rpm` handles `Obsoletes` — removing the old package and taking over its files.

APK does not auto-remove a package that was explicitly installed (`apk` reports `breaks: world[...]`). Existing APK installs must be migrated in two steps:

```shell
apk del minio && apk add --allow-untrusted ./aistor_<version>_<arch>.apk
```

Fresh APK installs of the renamed package work normally.
