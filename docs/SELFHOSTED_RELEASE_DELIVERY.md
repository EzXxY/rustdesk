# Self-hosted RustDesk Delivery Notes

## 本地交付目录
当前已从 `selfhosted-packages` release 下载并整理到本地目录：

- `rustdesk/Windows/`
- `rustdesk/Android/`
- `rustdesk/Linux/`
- `rustdesk/macOS/`
- `rustdesk/iOS/`

## 当前交付内容

### Windows
- `rustdesk-1.4.6-x86_64.exe`
- `rustdesk-1.4.6-x86_64.msi`
- `rustdesk-1.4.6-x86-sciter.exe`

### Android
- `rustdesk-1.4.6-aarch64.apk`
- `rustdesk-1.4.6-armv7.apk`
- `rustdesk-1.4.6-x86_64.apk`
- `rustdesk-1.4.6-universal.apk`

### Linux
- `rustdesk-1.4.6-aarch64.deb`
- `rustdesk-1.4.6-x86_64.deb`
- `rustdesk-1.4.6-armv7-sciter.deb`
- `rustdesk-1.4.6-x86_64-sciter.deb`
- `rustdesk-1.4.6-aarch64.AppImage`
- `rustdesk-1.4.6-x86_64.AppImage`
- `rustdesk-1.4.6-aarch64.flatpak`
- `rustdesk-1.4.6-x86_64.flatpak`
- `rustdesk-1.4.6-x86_64-sciter.flatpak`
- `rustdesk-1.4.6-0.aarch64.rpm`
- `rustdesk-1.4.6-0.aarch64-suse.rpm`
- `rustdesk-1.4.6-0.x86_64.rpm`
- `rustdesk-1.4.6-0.x86_64-suse.rpm`

### macOS
- `rustdesk-1.4.6-aarch64.dmg`
- `rustdesk-1.4.6-x86_64.dmg`

### iOS
- `rustdesk-1.4.6-aarch64-unsigned.ipa`

> 注意：当前 iOS 交付目标只是“产出 unsigned IPA 文件”，不包含签名、TestFlight、App Store 分发。

## 推荐使用的 GitHub Actions

### 1. 主线四端构建
- 文件：`.github/workflows/custom-4platform-unsigned.yml`
- 作用：更新源码后，先验证 Windows / Linux / Android / iOS 主线是否还能构建。

### 2. 主线桌面/主流 release 刷新
- 文件：`.github/workflows/custom-package-matrix.yml`
- 作用：把 Windows x64、Linux deb、macOS dmg 等主流桌面包汇总到 release。

### 3. 补缺包构建
- 文件：`.github/workflows/custom-rpm-flatpak-sciter-ipa.yml`
- 作用：补 x86 sciter、rpm、flatpak、ipa 这类缺口。

## 交付规则（经验固化）

1. **优先 artifact-first，再统一发布 release**
   - 不要让各平台 job 自己直接发 release，容易因为权限或 GitHub API 波动失败。
   - 先上传 artifact，再统一 `publish-collected` 发到 `selfhosted-packages`。

2. **Windows x86 Sciter**
   - 历史上 job 可能成功，但未必直接发布到 release。
   - 正确做法是从 32 位 Windows job 的 artifact 中抓真正的打包产物，再统一发布。

3. **RPM / SUSE RPM**
   - 由 Linux 主线 job 构建阶段生成。
   - 要单独添加 artifact 上传，否则只在 job 内部生成、最终 release 里看不到。

4. **IPA**
   - `flutter build ipa --no-codesign` 不一定直接产生 `.ipa`。
   - 当前已验证可以从 `Runner.xcarchive` 中取 `Runner.app`，组装 `Payload/Runner.app` 后手工封装成 unsigned IPA。

5. **release 名称**
   - 当前统一使用：`selfhosted-packages`
   - 适合长期追加包，不必每次新建 release。
