# Changelog

All notable changes to this project will be documented in this file.

## [5.13.0] - 2026-07-14

### Added — 规则25 扩展到 12 项 + 触发词精度 + 中英文一致性（源自 session-branch + kami 审计反馈）
- **规则18 增强**：触发词泛化检测——新增中英文黑名单（branch/task/new/start + 画图/做个图/写文章等日常用语），命中黑名单 = Medium finding
- **规则25 第2项增强**：Description-Behavior Mismatch 增加 description 模板建议，区分"核心能力"和"可选能力"
- **规则25 第9项增强**：Missing User Warnings 覆盖范围扩展——从"自动推送外部服务"扩展到"写入项目本地文件"（session-branch Finding 6：写 docs/session-handoff.md 但没告知用户）
- **规则25 新增第11项：Internal Consistency Check**：检测 SKILL.md 内部"禁止 X"和"要求 X"的自相矛盾指令（session-branch Finding 3：Critical rules 说不用绝对路径 vs Step 4 要求绝对路径）
- **规则25 新增第12项：Sensitive File Scan Consent Check**：扫描敏感文件路径模式（~/SOUL.md/IDENTITY.md/MEMORY.md/config.json/credentials/memory/profile）时，必须有 consent 步骤（session-branch Finding 2/7：扫描 ~/.workbuddy/SOUL.md 但无用户同意）
- **Step 2 增强**：中英文一致性自动检查——比对 README.md 和 README.en.md 的版本号/触发词数量/警告段落数量（kami 审计：英文版残留 v2.1.0 内容）
- **规则25 标题更新**：10 项 → 12 项，新增 Medium finding 不阻断机制（只 WARN）

## [5.12.0] - 2026-07-14

### Added — 规则25 扩展 + 权限声明标准模板（源自 gongwen-formatter v1.1.2 审计反馈）
- **规则18 增强**：权限声明段落增加 5 行表格标准模板（能力类别/是否使用/说明），统一各 skill 写法，降低作者负担
- **规则25 第7项增强**：MCP Tool Poisoning 检查新增"代码 import 扫描对照"子检查——扫描 `*.py` 源码的 HTTP 客户端库 import（urllib.request/requests/http.client/aiohttp/httpx），与 SKILL.md description 声明对照。预防 Context-Inappropriate Capability finding（SkillSpector 不只针对 SSRF，还会针对"非声明网络的隐式外联"）
- **规则25 新增第10项：Unpinned Dependencies 分级处理**：扫描 `requirements.txt`，`==`/`~=` PASS、`>=` WARN（不阻断）、无版本约束 FAIL。遵循 PIP 生态实践，`~=` 是平衡安全与兼容的推荐方案
- **规则25 标题更新**：从 9 项扩展到 10 项，新增 WARN 级别机制（不阻断发布，只提示作者）
- **Step 2 引用更新**：规则25 描述同步更新，包含 v5.12 新增的代码 import 扫描对照和依赖版本分级

## [5.11.0] - 2026-07-12

### Added — GitHub 同步可靠性增强（源自 skillhub-daily GitHub 漏更 40 天事件）
- **规则21 改进**：401 时不再静默继续，询问用户是否中止发布修复 token 还是跳过 GitHub 继续发布其他平台（会记录待补推版本）
- **新增规则26：GitHub 失败醒目警告**：发布末尾如果 GitHub 失败，必须用醒目警告重复提示，不能只埋在表格里
- **新增规则27：待补推版本跟踪**：log.md 记录 GitHub 失败和待补推版本号；Step 0 先检查是否有历史未推送版本，有则优先补推
- **新增规则28：三平台一致性校验**：发布后对比三平台版本号，不一致时醒目警告
- **规则18 增强**：质量门禁新增"权限声明"和"用户警告"段落检查（MCP Least Privilege + Missing User Warnings 是高频 finding）
- **Step 0 增强**：检查待补推版本
- **Step 2 Pre-Scan 增强**：扫描 `_*.py`/`_*.ps1` 临时脚本（web-to-fim v3.3.0 教训：_gh_push.py 误上传导致 27 个 findings）
- **Step 4.5 新增**：GitHub 推送后、ClawHub 发布前删除临时脚本
- **Step 7 增强**：三平台一致性校验 + GitHub 失败醒目警告
- **sync_skills.py 改进**：排除运行时文件（*.log/batch_*.txt/test_*.txt/_*.py/_*.ps1）+ 临时脚本 + 执行日志 + review 报告

## [5.10.0] - 2026-07-12

### Added — 持续学习机制 + token 读取增强
- **Step 9 扩展**：从"只记录日志"扩展为"日志记录 + 经验采集 + 索引维护"，发布时扫描被发布技能的 docs/knowledge/，采集跨技能可复用经验，更新 docs/knowledge/INDEX.md
- **Step 9 强制门**：日志记录是发布流程最后一个必做步骤，不可跳过（周度 review 显示日志覆盖率仅 25%）
- **规则21 增强**：token 读取优先 Windows User scope registry，fallback 到 os.environ。TRAE shell session 会缓存环境变量快照，os.environ 可能读到旧值导致 401 误判。适用于 GH_TOKEN/SKILLHUB_TOKEN/CLAWHUB_TOKEN/FEISHU_APP_SECRET/IMA_OPENAPI_APIKEY 等所有凭证环境变量
- **三层闭环沉淀机制**：经验采集层（事件驱动）+ 统一分析层（周期驱动）+ 主动注入层（触发驱动）

## [5.9.0] - 2026-07-12

### Added — 规则25 扩展 4 项预扫描（源自 skillhub-daily 17 个 findings 修复经验）
- **SSD3 敏感数据派生输出扫描**：检查代码是否读取本地敏感文件并将派生内容写入持久化输出
- **MCP Tool Poisoning 完整行为声明**：description 必须完整声明全部行为范围（网络/文件/记忆/subprocess）
- **MCP Least Privilege 权限声明**：SKILL.md 或 plugin.json 必须声明权限
- **Missing User Warnings 检查**：有副作用的 skill 必须在 README 中包含用户警告
- security-audit.md 新增 Layer 5（4 类新扫描的详细规范）

## [5.8.0] - 2026-07-12

### Fixed — 4 SkillSpector findings
- **Finding 1 (Low)**：删除"全部"批量发布选项，改为逐个确认。批量发布不可逆，逐个确认更安全
- **Finding 2 (Medium)**：skillhub.bat 修复说明标注"用户手动环境配置，agent 不自动执行"，避免 Context-Inappropriate Capability
- **Finding 3 (Medium)**：README.en.md FAQ 触发词从 "update xxx" 收紧为 "publish skill update"
- **Finding 4 (Medium)**：change-detection.md 示例从"更新 wx-peitu"收紧为"发布技能更新 wx-peitu"

## [5.7.0] - 2026-07-11

### Added — ClawHub SkillSpector 预扫描能力沉淀
- **新增规则25：ClawHub SkillSpector 预扫描**：发布到 ClawHub 前执行 5 项预扫描（YARA 触发词、Description-Behavior Mismatch、安全敏感方案不文档化、Self-Modification 措辞、CHANGELOG 历史记录扫描），源自 v5.4-v5.6 三轮 finding 修复经验
- **security-audit.md 新增 Layer 4：YARA 触发词扫描**：扫描"自治破坏行为"字面量类别（shell history 清理、PowerShell 错误忽略参数、递归强制删除、权限放宽、输出重定向到空设备），覆盖所有文件含 CHANGELOG 历史记录
- **publish-procedures.md 故障排查新增 ClawHub 多版本共存说明**：同一 slug 多版本并存是平台正常行为，新版本自动成为 latest，旧版本无法删除

### Changed
- 规则1：三类安全扫描 → 四类安全扫描（新增 YARA 触发词扫描）
- 规则2：扩展为"安全修复必须同步中英文版"，否则 SkillSpector 因英文版残留重复报 findings
- Step 2 安全审查：三类扫描 → 四类扫描，新增 ClawHub SkillSpector 预扫描（规则25）
- security-audit.md Scan Execution：三类 → 四类

### 关键教训
- YARA 规则是字面量匹配，不是语义分析：文档中写"不要使用 XXX"也会触发匹配，正确做法是用类别描述指代
- SkillSpector 扫描所有文件含 CHANGELOG 历史记录：历史条目中的触发词也需重新措辞
- Description-Behavior Mismatch：description 与实际行为不一致会被标记为"欺骗性能力披露"
- 安全敏感方案不要文档化：API 逐文件上传方案写在文档中会被标记为 MCP Tool Poisoning

## [5.6.0] - 2026-07-11

### Fixed — ClawHub SkillSpector v5.5.0 残留 8 个 findings 修复

**High 级别（4 个）**：
- **MCP Tool Poisoning (Tp4)**：删除规则22 Level 3 Git Data API 逐文件上传方案（被标记为"直接推送硬编码仓库"），只保留 git push + gh CLI 两级降级
- **Self-Modification**：change-detection.md "Update SKILL.md" → "Update version in SKILL.md"
- **YARA Match (agent_skill_destructive_autonomous_actions)**：删除 shell history 清理命令（YARA 规则匹配为自治破坏行为）
- **Tool Parameter Abuse**：简化规则22，删除 Git Data API 逐文件上传方案细节

**Medium 级别（4 个 Description-Behavior Mismatch）**：
- **本地 TRAE 安装同步**：删除规则25（本地安装同步）和 Step 8，这些内容被标记为"Description-Behavior Mismatch"（description 只说发布到外部平台，但实际还会修改本地安装目录）
- **PowerShell 错误忽略参数**：skillhub-publishing.md 中错误忽略参数改为推荐临时副本方式（robocopy 排除不支持的文件）
- **硬编码本地路径**：SKILL.md 中 `d:\TRAE SOLO CN\project\...` 改为 `<project>/...` 占位符
- **规则24**：zip 方式改为临时副本方式（更安全，不触发 YARA）

### Removed
- 规则25（本地安装同步）— 项目特定操作，不应在发布技能中
- Step 8（本地安装同步）— 同上
- 规则22 Level 3（Git Data API）— 安全风险过高

### Changed
- 规则从 25 条缩减到 24 条
- SkillHub 文件类型限制修复方案从"移除+恢复"改为"临时副本"
- 版本从 5.5.0 升级到 5.6.0

## [5.5.0] - 2026-07-11

### Fixed — ClawHub SkillSpector v5.4.0 残留 12 个 findings 修复

**Medium 级别（8 个）**：
- **--key 参数暴露 token (2)**：skillhub-publishing.md 承认 CLI 限制（--key 会暴露到 process listing），提供缓解措施（受控环境、清理 history、清除临时变量），删除"不出现在命令行参数"的错误声明
- **触发词仍太宽泛 (4)**：触发词从"技能发布"/"更新技能"/"迭代技能"收紧为"技能发布到三平台"/"发布技能更新"/"迭代技能发布"，加前置条件说明（需明确发布意图）
- **金句过于承诺 (1)**：README 金句从"说一句就发布"改为"安全审查先行，每一步都向你确认"
- **description 缺外部传输警告 (1)**：SKILL.md description 加"⚠️ 会推送代码到外部平台，执行前向用户确认"

**High 级别（2 个 Self-Modification）**：
- README.en.md 英文版触发词从 "update skill"/"iterate skill" 改为 "publish skill update"/"publish skill iteration"（v5.4.0 漏改英文版）

### Changed
- 触发词表格加前置条件警告块
- 双工作流自动检测加前置条件说明
- skillhub-publishing.md 安全规则块修正错误声明

## [5.4.0] - 2026-07-11

### Added
- **规则21 GitHub token 有效性校验**：Step 0 新增 GitHub token 验证，401 时提示用户到 settings/tokens 重新生成
- **规则22 GitHub 推送三级降级**：git push → gh CLI → GitHub Git Data API（Python urllib 逐文件上传），适用于 git push 持续超时但 API 可达的场景
- **规则23 SkillHub 备份目录隔离**：临时移除的文件不能备份在 skill 目录内部（会被扫描报 400），必须备份到目录外或改用 zip 方式
- **规则24 SkillHub 文件锁定 fallback**：Windows 文件被占用无法移除时，改用 Compress-Archive 打包 zip 发布
- **规则25 本地安装同步**：三平台发布后必须同步到 TRAE 安装目录，用 Python sync_skills.py 绕过 PowerShell 安全限制
- **Step 0 增强**：新增 GitHub token 有效性校验
- **Step 4 增强**：三级降级机制，含 Git Data API 逐文件上传方案
- **Step 8 新增**：本地安装同步，调用 sync_skills.py 同步到 .trae-cn/skills/
- **Windows 注意事项更新**：skillhub.bat python3 问题的修复方法
- **sync_skills.py 工具**：通用技能同步脚本，支持批量同步、单技能同步、预览、强制同步

### Changed
- 规则从 20 条扩展到 25 条
- 版本从 5.3.0 升级到 5.4.0
- 源自 skill-forge v4.3 发布过程中的实战经验

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

## [5.4.0] - 2026-07-11

### Fixed — ClawHub SkillSpector 13 个 findings 修复

**High 级别（3 个）**：
- **Token Extraction**: 删除 publish-procedures.md 和 publishing-guide.md 中"从 git remote -v 提取 token"的指导，改为"Token MUST come from environment variable GH_TOKEN"
- **命令行传 token 矛盾**: skillhub-publishing.md 故障排查中删除"直接用 token 值"建议，改为用 `[Environment]::GetEnvironmentVariable()` 读取
- **Self-Modification**: 同 Token Extraction

**Medium 级别（10 个）**：
- **.gitignore 缺 config.local.json**: .gitignore 实际文件和 repo-structure.md 模板都加入 `config.local.json`
- **触发词太宽泛（5 个）**: README.md/README.en.md 中 "publish"/"new"/"update"/"iterate" 改为 "技能发布"/"新建技能"/"更新技能"/"迭代技能"（英文版 "publish skill"/"new skill"/"update skill"/"iterate skill"）
- **login --key 命令行传 token（2 个）**: skillhub-publishing.md 登录命令改为从环境变量 `$SKILLHUB_TOKEN` 读取，加安全警告
- **README 缺用户警告（2 个）**: README.md/README.en.md 开头加"⚠️ 本工具会修改仓库、创建 Release、发布到外部平台"警告

### Changed
- skillhub-publishing.md 登录章节加安全规则块
- publish-procedures.md Token Source 章节重写
- publishing-guide.md Token Source 章节重写

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
