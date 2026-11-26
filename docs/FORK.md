# Fork 关系与本地差异说明

## 与上游的关系

- 上游仓库：`Liquid-co/OpenSave`（GitHub 标记为 parent / source）
- 本仓：`Janlhin/OpenSave`，公开仓库，默认分支 `main`
- 本仓是上游的 fork，产品代码、文档与既有工作流以上游为准；本批治理改动只落在治理层（`docs/`、`.github/`、根目录治理文件、`tools/`），不改上游核心代码、构建配置与既有工作流。

## 本仓相对上游的差异

可核对的事实（2026-11-26，本地 `--depth 1` 克隆）：

- 本仓 `main` HEAD：`f9bb679b4a13403ed8c2c3f54cc99caa3f3a350b`，提交信息 "Website: relay saves sealed end to end from 2.4.0"，提交时间 2026-09-30。
- 本仓另有分支 `i18n/zh-nothing-running`，最近推送 2026-11-24。
- 拉取上游 `main` 浅快照后执行 `git rev-list --left-right --count upstream/main...HEAD` 为 `1  1`；浅克隆下完整合并基点未核对，历史提交的先后与分叉范围**未核对**。
- 文件树观察：本仓 `internal/` 下模块布局（如 `api/`、`cliapp/`、`cloud/`、`e2ee/`、`transfers/` 等）与上游 `main` 快照存在差异；具体逐文件改动清单未核对，不在此推断。

本批治理分支上新增的内容（仅治理层，不属于上游产品代码）：

- `docs/FORK.md`：本文件。
- `.github/pull_request_template.md`：PR 填写模板。
- `.github/dependabot.yml`：仅跟踪 `github-actions` 生态的版本更新；Go / npm / Python 依赖更新交给上游。
- `.github/workflows/verify.yml`：只检查本仓治理文件是否齐全、结构是否完整。
- `tools/check-governance.py`：治理文件自检脚本（纯标准库）。

## 既有 CHANGELOG 口径

根目录 `CHANGELOG.md` 是上游的发布说明（SemVer 条目），本仓不回写、不改写。本仓治理层改动记录在对应 PR 描述与本文件中，不进入上游发布说明。

## 禁止本地魔改的目录与文件

以下内容来自上游，本地分支不得修改、不得格式化重排：

- `cmd/`、`internal/`、`e2e/`、`relay/`、`opensave-decky-plugin/`、`packaging/`、`scripts/`、`testutil/`：上游代码、插件、打包与测试目录。
- `docs/`：上游既有文档与网站资源。
- `go.mod`、`go.sum`、`Dockerfile`、`changelog.go`：上游清单、锁文件与构建文件。
- `README.md`、`GETTING_STARTED.md`、`USER_GUIDE.md`、`PRIVACY.md`、`LICENSE`、`CHANGELOG.md`：上游既有正文与许可。
- `.github/workflows/ci.yml`、`.github/workflows/release.yml`：上游既有工作流，本批不改动、不复刻。

如确需改动上述内容，应先向上游提交并在本仓同步，而不是在 fork 内单独魔改。

## 同步上游的步骤

1. 添加上游远程（本地备份中已添加）：

   ```bash
   git remote add upstream https://github.com/Liquid-co/OpenSave.git
   ```

2. 拉取上游最新 `main`：

   ```bash
   git fetch upstream main
   ```

3. 在治理分支上合并或变基上游：

   ```bash
   git merge upstream/main   # 或 git rebase upstream/main
   ```

4. 冲突只允许出现在治理文件上；若上游核心目录出现冲突，先核对是否误改。

5. 上游检查命令以上游工作流与文档为准（Go vet / test、前端 `npm ci` / `test` / `build`、Decky 插件检查）；这些不在本仓治理 CI 中重复运行。

6. 推送前确认 `git diff --stat upstream/main` 中不包含被禁止魔改的目录。

## 上游 CI 排查记录（2026-11-26）

排查结论：**触发条件与仓库设置均正常，未做配置修复。**

- Actions 设置：`enabled: true`，`allowed_actions: all`，未禁用。
- `ci.yml` 触发条件：`push` 全分支（`branches: ["**"]`）+ `pull_request`，无 `paths` 过滤。
- `release.yml` 触发条件：推送 `v*` 标签 + `workflow_dispatch`。
- 观测记录：`ci.yml` 在功能分支 `i18n/zh-nothing-running` 上完成并成功（run `45455779501`，约 12m34s，2026-11-24 18:05:55Z）。
- `main` 上未见运行记录的原因：`main` 自 2026-09-30 起没有新推送，缺少 push / PR 触发事件；这不是工作流配置缺陷。
- `release.yml` 未见运行：仓库尚无 `v*` 标签推送，属预期行为。
- 本批未改动上述两个上游工作流；上游工作流中引用的 action 版本与第三方 action 不在本仓治理范围内调整。
