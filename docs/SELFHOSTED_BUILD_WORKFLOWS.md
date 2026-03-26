# Self-hosted Build Workflows

这个文档记录当前仓库里用于**自建 RustDesk 客户端打包**的几条关键 GitHub Actions。

## 1. custom-4platform-unsigned.yml

**文件**：`.github/workflows/custom-4platform-unsigned.yml`

**用途**：主线 4 端构建验证。

**目标**：
- Windows
- Linux
- Android
- iOS（IPA 文件构建链）

**建议使用场景**：
- RustDesk 源码更新后，先确认主线 4 端还能正常出包。
- 更偏“先验证构建没炸”。

---

## 2. custom-package-matrix.yml

**文件**：`.github/workflows/custom-package-matrix.yml`

**用途**：桌面主线成品汇总发布。

**目标**：
- Windows x64 exe/msi
- Linux deb
- macOS dmg

**建议使用场景**：
- 主线版本稳定后，把最常用桌面包统一刷新到 release。

---

## 3. custom-rpm-flatpak-sciter-ipa.yml

**文件**：`.github/workflows/custom-rpm-flatpak-sciter-ipa.yml`

**用途**：补齐 release 里缺的兼容/发行矩阵。

**目标**：
- Windows x86 Sciter
- rpm / suse rpm
- flatpak
- IPA 文件

**建议使用场景**：
- 当主流包已齐，只需要补缺失的兼容包时使用。

> IPA 当前目标只是**构建出 IPA 文件**并交付，不包含 iOS 正式签名/上架/TestFlight 分发。

---

## 推荐顺序

1. 先跑 `custom-4platform-unsigned.yml`
2. 再跑 `custom-package-matrix.yml`
3. 最后按需跑 `custom-rpm-flatpak-sciter-ipa.yml`

这样最稳，不容易一条 workflow 同时承担太多目标导致失败。
