# 简记 · macOS 桌面端

家庭记账工具「简记」的 macOS 桌面端，基于 **Tauri v2 + SQLite** 构建，数据完全存本机。

> 本仓库 `YU06118/jianji-macos` 由 Windows 桌面端仓库 [`YU06118/jianji-desktop`](https://github.com/YU06118/jianji-desktop) 拆分而来，版本 **0.1.8**，产品名「简记」，应用标识符 `com.jianji.ledger`。
> Windows 安装包（NSIS/MSI）继续由 `jianji-desktop` 仓库发布，本仓库不再构建 Windows 产物；微信小程序端见 [`YU06118/jianji-miniprogram`](https://github.com/YU06118/jianji-miniprogram)。

## 平台支持

| 项 | 说明 |
|---|---|
| 最低系统 | macOS 11.0（Big Sur） |
| 构建架构 | Intel `x86_64-apple-darwin` 与 Apple Silicon `aarch64-apple-darwin`，分别出包 |
| 打包目标 | `app` + `dmg`（`bundle.targets: ["app", "dmg"]`），并生成更新用 `.app.tar.gz` 与 `.sig` |
| 代码签名 | **未签名、未公证**（`signingIdentity: null`），首次打开需手动放行 |
| 自动更新 | GitHub Releases + minisign 签名，推送 tag 自动双架构发版 |

## 功能概览

- 记一笔：系统输入法金额输入、支出/收入、分类、备注、日期、小票图片
- 明细：按日期分组、关键词搜索、筛选 chips、编辑/删除、回收站
- 统计：周/月/年/多年维度、真实数据三卡、分类对比、热力图
- 分类管理：新建/编辑/删除，事件委托绑定
- 每日记账提醒：自定义时分（仅应用运行时有效）
- 数据：CSV 导出 + JSON 备份/导入；浏览器环境降级走 localStorage
- 热更新：GitHub Releases + minisign 签名，tag 推送自动发版
- 外观：跟随系统 / 浅色 / 深色三档

## 快速开始

前置依赖：macOS 11+、Xcode Command Line Tools（`xcode-select --install`）、Node.js 18+、Rust（[rustup](https://rustup.rs)）。

```bash
cd desktop
npm install

# 本机开发调试
npm run tauri dev

# 分别构建两个架构（可在同一台 Mac 上交叉构建）
npm run tauri build -- --target x86_64-apple-darwin
npm run tauri build -- --target aarch64-apple-darwin
```

产物（以 aarch64 为例，Intel 替换目录名即可）：

- `desktop/src-tauri/target/aarch64-apple-darwin/release/bundle/dmg/*.dmg` — 分发包
- `desktop/src-tauri/target/aarch64-apple-darwin/release/bundle/macos/*.app` — 应用本体
- `.../bundle/macos/*.app.tar.gz` 与 `*.app.tar.gz.sig` — 自动更新用压缩包与 minisign 签名

### 首次打开（未签名应用）

- Finder 中右键「简记.app」→「打开」→ 在提示框再点「打开」；
- 或在终端执行：`xattr -dr com.apple.quarantine "/Applications/简记.app"`；
- 应用内自动更新要求「简记.app」位于 `/Applications` 下，否则更新安装会失败。

## 数据目录

```
~/Library/Application Support/com.jianji.ledger/
├── jianji.db      SQLite 数据库（记录、分类、设置）
└── images/        小票图片
```

JSON 备份只包含记录与分类，不含图片文件本体；迁移时如需保留小票原图，需一并拷贝 `images/`。

## 自动更新与发版

- 更新端点：`https://github.com/YU06118/jianji-macos/releases/latest/download/latest.json`
- `latest.json` 平台键：`darwin-x86_64`（Intel）与 `darwin-aarch64`（Apple Silicon），各自指向对应的 `jianji_<version>_<arch>.app.tar.gz` 与签名。
- CI：`.github/workflows/macos-release.yml`
  - 推送 `v*` 标签：并行构建两个架构 → 签名 → 创建 Release，上传 2 个 DMG、2 个 `.app.tar.gz`、2 个 `.sig` 与 `latest.json`（资产名统一为 ASCII，如 `jianji_0.1.8_x64.dmg`）。
  - `workflow_dispatch`：只构建并上传 Actions artifacts，不发布 Release。
- 签名密钥复用现有 minisign 密钥，仓库 Secrets：
  - `TAURI_SIGNING_PRIVATE_KEY`：`~/.tauri/jianji.key` 的内容（已配置）；
  - `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`：该密钥为空密码，可留空。
- 发版步骤：改三处版本号（`desktop/src-tauri/tauri.conf.json`、`desktop/src-tauri/Cargo.toml`、`desktop/package.json`）→ 提交 → `git tag vX.Y.Z` 并推送 tag。

## 目录结构

```
desktop/
├── frontend/              原生 HTML/CSS/JS 前端
├── src-tauri/             Rust 后端（Tauri 命令、SQLite、通知、更新）
│   ├── src/main.rs        17 个 Tauri 命令 + 插件注册 + 通知定时器
│   └── icons/icon.icns    macOS 应用图标
shared/                    双端共享数据契约与核心算法（仅参照/单测，运行时内置）
docs/                      API、更新日志、部署、热更新、交接文档
prototypes/                桌面端视觉原型（浏览器直接打开）
.github/workflows/         macos-release.yml（tag 触发的双架构签名发版流水线）
```

## 文档索引

- `docs/使用与部署说明.md` — macOS 环境、构建、安装、Gatekeeper、数据迁移
- `docs/API.md` — Tauri 命令层、设置项、共享核心接口
- `docs/CHANGELOG.md` — 版本演进
- `docs/热更新设计与使用说明.md` — 更新机制、安全模型、发布 SOP、回滚
- `docs/GitHub上传流程.md` — 日常上传与 tag 自动发版
- `docs/项目进度交接文档-2026-11-21.md` — 现状、待办、架构要点、已知问题（Windows 基线）
- `docs/交付清单.md` — 交付内容总览

## 安全与数据

数据本地优先：`jianji.db`（SQLite）+ `images/`，位于 `~/Library/Application Support/com.jianji.ledger/`。除自动更新检查外无网络请求。

应用未签名、未公证：minisign 签名只保证更新包完整性与来源，不解决 Gatekeeper 信任问题；首次打开需按上文手动放行。

## 许可与作者

个人项目，作者 YU06118。
