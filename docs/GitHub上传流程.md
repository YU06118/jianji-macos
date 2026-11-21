# 简记 · GitHub 仓库说明与上传流程

> 仓库：https://github.com/YU06118/jianji-macos ｜ 主分支：`main` ｜ 更新：2026-11-21

## 一、仓库现状

| 内容 | 位置 |
|---|---|
| macOS 桌面端（Tauri v2 + SQLite + 热更新） | `desktop/` |
| 共享数据契约与核心逻辑（26 项单测） | `shared/` |
| 桌面端视觉原型（自包含可预览） | `prototypes/` |
| 文档（需求/部署/热更新/GitHub） | `docs/` |
| 发布流水线（tag 触发双架构构建、签名、发布） | `.github/workflows/macos-release.yml` |

Windows 桌面端源码与安装包在 `YU06118/jianji-desktop`；微信小程序端在 `YU06118/jianji-miniprogram`。

同步约定：**每完成一个文件/模块即提交并推送一次**。

## 二、提交约定

| 前缀 | 含义 |
|---|---|
| feat(x) | 新功能（x = shared / desktop / prototypes / docs / ci） |
| fix(x) | 缺陷修复 |
| test(x) | 测试与自检 |
| chore / docs / ci | 版本号 / 文档 / 流水线 |

## 三、日常上传流程（手动）

```bash
cd C:\Users\TDavid\WorkBuddy\jianji-macos
git add <文件>
git commit -m "docs: 变更说明"
git push origin main          # 网络抖动（502/CONNECT tunnel）时稍候重试即可
```

凭据：Windows 凭据管理器已存 GitHub 账号 **YU06118** 的令牌，git push 免密。

## 四、发版上传流程（自动，推荐）

发新版只需两步，其余全部由 CI 完成：

```bash
# 1) 改版本号三处：desktop/src-tauri/tauri.conf.json、desktop/src-tauri/Cargo.toml、desktop/package.json
git add . && git commit -m "chore: bump x.y.z"
# 2) 打 tag 推送（CI 由 v* 标签触发）
git tag vX.Y.Z && git push origin main vX.Y.Z
```

CI（`.github/workflows/macos-release.yml`，约 15–25 分钟）自动完成：
1. `macos-14` 上按矩阵分别构建 `x86_64-apple-darwin`（Intel）与 `aarch64-apple-darwin`（Apple Silicon）
2. 使用 Secrets 中的 minisign 私钥签名，产出 2 个 DMG、2 个 `.app.tar.gz`、2 个 `.sig`
3. 资产重命名为 ASCII（`jianji_x.y.z_x64.*` / `jianji_x.y.z_aarch64.*`）
4. 生成 `latest.json`（含 `darwin-x86_64` 与 `darwin-aarch64`），创建/复用 Release 并上传全部资产

手动触发 `workflow_dispatch` 只构建并上传 Actions artifacts，**不创建 Release**。

验收检查：
- `https://github.com/YU06118/jianji-macos/releases/latest/download/latest.json` 可访问，且 `platforms["darwin-x86_64"]` 与 `platforms["darwin-aarch64"]` 均存在
- 在 `/Applications` 中已安装旧版，设置 →「检查更新」能走完整更新流程
- 两个 DMG 均可挂载，首次打开按「右键 → 打开」可绕过 Gatekeeper

## 五、Secrets 清单（仓库 Settings → Secrets and variables → Actions）

| Secret | 用途 | 状态 |
|---|---|---|
| `TAURI_SIGNING_PRIVATE_KEY` | 更新包 minisign 签名私钥（`~/.tauri/jianji.key` 的内容） | ⬜ 待配置 |
| `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` | 私钥密码 | ⬜ 待配置 |

> 两个 Secret 任一缺失时构建出的更新包无法被客户端验签，自动更新会失败。私钥文件本体在 `~/.tauri/jianji.key`，**丢失将无法签发后续更新**，务必备份。本仓库不配置 COS 镜像同步。

## 六、不入库内容（.gitignore）

- `desktop/node_modules/`、`desktop/src-tauri/target/`、`desktop/src-tauri/gen/`
- `.workbuddy/`（会话数据）、系统临时文件
- **签名私钥永不入库**（在本机用户目录，且建议另行备份到离线介质）
