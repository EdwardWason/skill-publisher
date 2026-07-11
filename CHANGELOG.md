# Changelog

All notable changes to this project will be documented in this file.

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
