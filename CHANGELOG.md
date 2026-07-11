# Changelog

All notable changes to this project will be documented in this file.

## [5.1.0] - 2026-07-11

### Added
- **platform**: 新增 SkillHub 平台同步发布支持 — 三平台同步（GitHub + ClawHub + SkillHub）
- **frontmatter**: SKILL.md frontmatter 兼容 SkillHub 5 字段（slug/displayName/version/summary/license），与 ClawHub 字段共存
- **publish**: SkillHub dry-run 预检 — `skillhub publish <path> --dry-run` 发布前格式检查
- **security**: 凭证扫描新增 `skh_` 前缀（SkillHub API Token）
- **rules**: 规则从 12 条扩展到 15 条（新增规则 13-15：SkillHub frontmatter、dry-run 预检、SKILLHUB_TOKEN 保护）
- **workflow**: 执行流程从 6 步扩展到 7 步（新增 Step 6 SkillHub 发布）
- **docs**: 新建 `references/skillhub-publishing.md` — SkillHub CLI 安装、登录、frontmatter 兼容、dry-run、Windows 兼容、故障排查
- **troubleshooting**: 新增 6 个 SkillHub 故障案例（command not found、exit code 9009、401/403/409/429 错误）

### Changed
- **SKILL.md**: frontmatter 新增 slug/displayName/version/summary/license 字段
- **SKILL.md**: description 更新为"三平台同步推送 GitHub + ClawHub + SkillHub"
- **SKILL.md**: 输出格式发布结果表格新增 SkillHub 行
- **platform**: 从双平台（GitHub + ClawHub）升级为三平台（GitHub + ClawHub + SkillHub）

### Windows Compatibility
- **skillhub CLI**: Windows 上 `python3` 是 Store stub（exit code 9009），需用 `python` 或 `C:\Python313\python.exe` 直接调用 `skills_store_cli.py`
- **PowerShell**: `$env:SKILLHUB_TOKEN` 在参数传递时可能被吞，login 命令需直接用 token 值

## [5.0.0] - 2026-07-11

### Breaking Changes
- **security**: 删除 `upload_to_github.py`（含真实 GitHub Token 和本地绝对路径，安全红线违规）

### Added
- **security**: 凭证扫描模式扩展 — 新增 `cli_|IMA_OPENAPI|FEISHU_APP|APP_SECRET|CLIENTID|APIKEY|client_id|client_secret`（源自 IMA/飞书凭证泄露事件）
- **publish**: 版本号查重新增 — ClawHub 发布前必须 `clawhub inspect <slug>` 查重版本号，重复则递增 PATCH
- **publish**: ClawHub 自动生成文件排除规则 — `skill-card.md`、`.clawhub/`、`_meta.json` 禁止发布
- **publish**: frontmatter description 与 Short summary 同步规则 — 更新 description 必须递增版本号重新发布
- **rules**: 规则从 7 条扩展到 12 条（新增规则 9-12）
- **troubleshooting**: 新增 5 个故障案例（skill-card.md 被拒、版本号重复、Short summary 未更新、__pycache__ 打包、凭证泄露）
- **gitignore**: .gitignore 模板新增 `__pycache__/`、`*.pyc`、`*.pyo`、`.clawhub/`、`skill-card.md`、`_meta.json`

### Changed
- **SKILL.md**: 执行流程从 5 步扩展到 6 步（新增 Step 3 版本号查重）
- **SKILL.md**: frontmatter description 更新 — 新增"版本号查重"、"ClawHub 自动文件排除"关键词
- **SKILL.md**: 输出格式从 3 部分扩展到 4 部分（新增"版本号查重结果"）
- **triggers**: 新增触发词"更新技能"、"迭代技能"

### Fixed
- **security**: 修复 upload_to_github.py 硬编码 GitHub Token 的安全漏洞
- **security**: 修复 upload_to_github.py 硬编码本地绝对路径 `d:\TRAE SOLO CN\project\...` 的隐私泄露

## [4.0.0] - 2026-06-11

### Breaking Changes
- **workflow**: Added Workflow B (iteration update) alongside Workflow A (new publish), changing the skill from single-workflow to dual-workflow

### Added
- **workflow-b**: Change detection with file category mapping and version bump decision tree
- **workflow-b**: Auto CHANGELOG generation from git log with Conventional Commits parsing
- **workflow-b**: Version bump with three-file sync (SKILL.md → plugin.json → CHANGELOG.md)
- **workflow-b**: Release update flow for PATCH versions on same day
- **publish**: gh CLI integration as Method B in push fallback chain (git push → gh CLI → REST API)
- **readme**: Product landing page 21-chapter standard with smart adaptation by Skill complexity
- **readme**: Bilingual independent files strategy (README.md + README.en.md)
- **readme**: Golden quote opening, negation contrast, navigation links, limitations section
- **community**: config.yml for Discussions redirect in .github/ISSUE_TEMPLATE/
- **skill**: Provenance block (quote + HTML comment) for source identification
- **skill**: Version sync rule — SKILL.md → plugin.json → CHANGELOG.md consistency check
- **skill**: Smart README adaptation — simple Skills (≤5 rules) get required chapters only
- **skill**: SKILL.md content extraction before README generation
- **references**: change-detection.md — file category mapping, bump decision tree, Conventional Commits guide
- **references**: changelog-generation.md — git log extraction, commit parsing, CHANGELOG-to-Release-Notes conversion

### Changed
- **rules**: Expanded from 12 to 15 rules (added rules 13-15 for Workflow B)
- **steps**: Expanded from 13 to 25 steps (Workflow A: 1-13, Workflow B: 14-25)
- **publish**: Push fallback chain changed from 2-level (git push → REST API) to 3-level (git push → gh CLI → REST API)
