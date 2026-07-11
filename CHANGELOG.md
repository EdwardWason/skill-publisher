# Changelog

All notable changes to this project will be documented in this file.

## [5.2.0] - 2026-07-11

### Added
- **allowed-tools**: frontmatter 新增工具白名单声明（Bash(git/clawhub/skillhub/gh/python) + Read/Write/Edit/Glob/Grep），符合 TRACE T维度最小权限原则
- **前置条件校验**（规则17）：发布前校验4项前置条件（目录存在/SKILL.md存在/平台登录态/Git配置），任何一项不满足 = 中止发布并明确告知用户缺什么、怎么修
- **Skill 质量门禁**（规则18）：发布前快速检查 Skill 质量（SKILL.md行数/description存在性/description长度/Do NOT范围声明），不达标 = 拒绝发布并建议先修复
- **Step 0**: 执行流程新增 Step 0 前置条件校验，在 Step 1 仓库结构生成之前执行
- **示例模块**: 新增5个完整示例，覆盖1个常见输入 + 4个边界/异常输入（目录不存在/SKILL.md缺失/质量不达标/未登录），符合 TRACE R维度异常处理反馈要求

### Changed
- 规则从 16 条扩展到 18 条
- 版本从 5.1.0 升级到 5.2.0
- 源自 skill-forge v4.2 TRACE 评测体系整合后的验证检查

## [5.3.0] - 2026-07-11

### Added
- **rule 19**: 复杂输入处理 — 未指明 Skill / 多 Skill / 路径模糊时先确认目标（TRACE R维度）
- **rule 20**: SkillHub 发布前 TRACE 五维度预检 — T(信任)/R(可靠)/A(适用)/C(规范)/E(有效)，任何维度 FAIL 中止 SkillHub 发布
- **examples 6-8**: 复杂输入示例（未指明/路径模糊/TRACE 预检失败）
- **Step 6**: 加入 TRACE 预检步骤（规则20）

### Changed
- 规则从 18 条扩展到 20 条
- summary 加入 "TRACE 预检" 关键词
- SkillHub 发布流程从 5 步扩展到 6 步（新增 TRACE 预检）

### Fixed
- 修复 SkillHub 上仍显示 v5.1.0 的问题（v5.2.0 未推送三平台）
- 修复 v5.2.0 已加 allowed-tools 和示例但未发布的问题

## [5.2.0] - 2026-07-11

### Added
- **rule 17**: 前置条件校验（目录存在/SKILL.md存在/平台登录态/Git配置）
- **rule 18**: Skill 质量门禁（SKILL.md行数/description/Do NOT声明）
- **examples 1-5**: 常见输入 + 4 种边界/异常输入示例
- **allowed-tools**: frontmatter 声明工具白名单
- **Step 0**: 前置条件校验步骤

### Changed
- 规则从 16 条扩展到 18 条
- 执行流程从 7 步扩展到 8 步（新增 Step 0）

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
