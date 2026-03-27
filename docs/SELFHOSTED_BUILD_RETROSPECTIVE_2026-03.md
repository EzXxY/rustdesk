# Self-hosted RustDesk Build Retrospective (2026-03)

## 背景目标
目标是把 RustDesk 改造成：
- 默认走自建 rendezvous / key
- 尽量避开官方更新/API fallback/public STUN 默认值
- 能通过 GitHub Actions 稳定构建并把成品集中放到 release

## 关键结论

### 1. 只改 rustdesk 主仓库注入参数不够
最初尝试只在 `rustdesk` 主仓库做 `RENDEZVOUS_SERVER / RS_PUB_KEY` 注入，运行测试表明不够彻底。
最终正确方案是：
- 同步 `EzXxY/hbb_common` 到上游最新
- 直接改 `hbb_common` 默认 `RENDEZVOUS_SERVERS` 和 `RS_PUB_KEY`
- 再让 `EzXxY/rustdesk` 子模块指向 `EzXxY/hbb_common`

### 2. “边构建边发布 release” 不稳定
多次失败说明：
- 平台 job 直接 `Publish Release` 很容易因为权限/重试/GitHub API 抽风失败
- 更稳的方式是：
  1. 平台 job 先产出 artifact
  2. 最后单独一个 `publish-collected` job 统一往 release 发

### 3. 要分层拆 workflow
一条超大 workflow 很容易因为一个目标挂掉而整条失败。
后续拆成：
- 主线四端构建
- 主线桌面 release 刷新
- 补缺包 release
- IPA 专线调试/交付

### 4. IPA 是“构建文件交付”和“最终可安装分发”两回事
已验证：
- iOS 构建链可以跑通
- `flutter build ipa --no-codesign` 不一定直接得到 `.ipa`
- 但 `Runner.xcarchive` 可以拿来手工组出 unsigned IPA

所以当前交付边界明确为：
- **只交付 unsigned IPA 文件**
- 不处理签名、TestFlight、App Store

## 这次折腾中容易犯的错

1. 误把历史 artifact 里的裸 `rustdesk.exe` 当成 `x86-sciter.exe` 正式成品
2. 把 release 页面现状和 Actions artifact 现状混为一谈
3. 试图在一次 workflow 里同时满足“验证构建”“补缺口”“直接发 release”三个目标，导致排错困难
4. 直接硬改底层网络逻辑，影响了多平台构建稳定性

## 固化经验

- **默认 server/key 必须优先改 `hbb_common`**
- **release 统一由 artifact 汇总产生**
- **IPA 单独处理，不混在大矩阵里排错**
- **缺什么补什么，不要每次重新打一整套**
