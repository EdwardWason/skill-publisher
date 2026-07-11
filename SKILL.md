---
name: "skill-publisher"
description: "技能发布 — 将已有 Skill 三平台同步推送到 GitHub + ClawHub + SkillHub。当用户说 技能发布/发布技能/更新技能/迭代技能 时触发。含安全审查、隐私清洗、版本号查重、仓库结构生成、ClawHub 自动文件排除、SkillHub dry-run 预检。Do NOT use for creating skill content, general coding, or non-skill projects."
slug: skill-publisher-ai
displayName: Skill Publisher
version: 5.3.0
summary: 三平台同步发布技能到 GitHub + ClawHub + SkillHub，含安全审查、版本号查重、TRACE 预检、dry-run。
license: MIT
allowed-tools: "Bash(git:*), Bash(clawhub:*), Bash(skillhub:*), Bash(gh:*), Bash(python:*), Bash(cat:*), Bash(ls:*), Bash(mkdir:*), Bash(cp:*), Bash(mv:*), Bash(rm:*), Bash(Compress-Archive:*), Read, Write, Edit, Glob, Grep"
---

# 技能发布

将已有 Skill 三平台同步推送到 GitHub + ClawHub + SkillHub，含安全审查、隐私清洗、版本号查重、标准仓库结构生成、ClawHub 自动文件排除、SkillHub dry-run 预检。

## 何时触发

**当用户说出以下词汇时，立即调用本技能：**
- "技能发布"
- "发布技能"
- "更新技能"
- "迭代技能"

**注意**：如果用户说"技能熔炉"，应触发 skill-forge（全流程），不是本技能。

## 与技能熔炉的关系

本技能是技能熔炉（skill-forge）的独立触发入口，只执行 Phase 3 发布流程。完整流程（创建→评估→发布）请使用技能熔炉。

**详细文档共享**：本技能读取 skill-forge 的 `references/publishing-guide.md`，内容完全一致。

## 任务
只做 Skill 的发布准备与推送：生成标准仓库结构 → 安全审查 → 隐私清洗 → 版本号查重 → 推送 GitHub → 发布 ClawHub。不做 Skill 内容创建、不做代码开发。

## 输出格式
### 一、仓库结构生成报告
列出所有生成/更新的文件及路径

### 二、安全审查结果
| 审查项 | 状态 | 详情 |
|--------|------|------|
| 凭证泄露 | PASS/FAIL | 扫描结果 |
| 本地路径 | PASS/FAIL | 扫描结果 |
| 危险命令 | PASS/FAIL | 扫描结果 |
| 分发物判定 | PASS/FAIL | 多余文件列表 |

### 三、版本号查重结果
| ClawHub 已发布版本 | 待发布版本 | 状态 |
|-------------------|-----------|------|
| vX.Y.Z ... | vX.Y.Z | 可发布/版本号冲突 |

### 四、发布结果
| 平台 | 地址 | 版本 | 状态 |
|------|------|------|------|
| GitHub | URL | vX.Y.Z | 成功/失败 |
| ClawHub | slug | vX.Y.Z | 成功/失败 |
| SkillHub | slug | vX.Y.Z | 成功/失败 |

## 规则
1. 发布前必须执行三类安全扫描，任何 FAIL = 阻止发布
2. README 必须中英双语，Badge 用中文标签
3. ClawHub 发布前必须先 `clawhub inspect <slug>` 检查 slug 占用
4. **ClawHub 发布前必须查重版本号**：`clawhub inspect <slug>` 查看已发布版本列表，待发布版本号不能与已发布版本重复，重复则递增 PATCH
5. Windows 环境禁止使用 heredoc 语法
6. git push 失败时降级为 gh CLI，再降级为 GitHub REST API 逐文件上传
7. --tags 只能用 ASCII 字符（中文会报错）
8. 向 GitHub API 发送中文 JSON 必须用 Python（PowerShell 会损坏中文）
9. **凭证扫描必须覆盖新模式**：除原模式外，还需扫描 `cli_|IMA_OPENAPI|FEISHU_APP|APP_SECRET|CLIENTID|APIKEY|client_id|client_secret`（2026-07 新增，源自 IMA/飞书凭证泄露事件）
10. **ClawHub 自动生成文件必须排除**：`skill-card.md`、`.clawhub/` 目录由 ClawHub 自动生成，禁止发布（2026-07 新增，源自 skill-card.md 发布被拒事件）
11. **frontmatter description 决定 ClawHub Short summary**：更新 description 后必须重新发布才能同步 Short summary；首次发布后 description 不会自动更新，必须递增版本号重新发布（2026-07 新增，源自 Short summary 未更新事件）
12. **.gitignore 必须排除 Python 缓存**：`__pycache__/`、`*.pyc`、`.clawhub/` 必须在 .gitignore 中（2026-07 新增，源自 __pycache__ 打包事件）
13. **SkillHub frontmatter 必须包含 5 字段**：`slug`（全网唯一）、`displayName`、`version`、`summary`、`license`，与 ClawHub 的 name/description 共存于同一 frontmatter（2026-07 新增，支持 SkillHub 平台）
14. **SkillHub 发布前必须 dry-run 预检**：`skillhub publish <path> --dry-run` 检查格式，通过后才能正式发布（2026-07 新增，源自 SkillHub CLI 规范）
15. **SKILLHUB_TOKEN 不可硬编码**：token 只通过环境变量 `SKILLHUB_TOKEN` 传递，安全扫描必须检查 `skh_` 前缀的硬编码值（2026-07 新增，支持 SkillHub 平台）
16. **SkillHub 发布前必须临时移除不支持的文件类型**：`.gitignore`、`LICENSE`（无扩展名）、`.claude-plugin/`、`.github/` 会被 SkillHub 拒绝（400 错误）。发布前备份并移除，发布后立即恢复。ClawHub 和 GitHub 不受此限制（2026-07 新增，源自 SkillHub 文件类型限制）
17. **前置条件校验**（v5.2 新增，TRACE R维度）：开始发布前必须校验4项前置条件，任何一项不满足 = 中止发布并明确告知用户：
    - **目录存在**：指定路径必须存在且非空，否则报"目录不存在或为空：`<path>`，请确认 Skill 路径"
    - **SKILL.md 存在**：目录下必须有 SKILL.md 文件，否则报"未找到 SKILL.md，这不是一个有效的 Skill 目录"
    - **平台登录态**：`clawhub whoami` 和 `skillhub auth whoami` 必须通过，否则报"`<平台>` 未登录，请先执行 `<登录命令>`"
    - **Git 配置**：`git config user.name` 和 `git config user.email` 必须有值，否则报"Git 用户信息未配置，请先执行 `git config` 设置"
18. **Skill 质量门禁**（v5.2 新增，TRACE R维度）：发布前快速检查 Skill 质量，以下任一情况 = 拒绝发布并建议先修复：
    - SKILL.md 超过 300 行 → 报"SKILL.md 过长（`<N>`行），建议精简到 200 行以内再发布"
    - frontmatter 缺少 `description` → 报"description 缺失，无法自动触发，请先补全"
    - description 超过 250 字符 → 报"description 过长会被截断，核心触发词需在前 200 字符内"
    - 无 `Do NOT` 范围声明 → 报"description 缺少 Do NOT 范围声明，可能导致误触发"
19. **复杂输入处理**（v5.3 新增，TRACE R维度）：当用户未指明发布哪个 Skill，或工作目录下存在多个 Skill 时，必须先确认目标：
    - **未指明**：用户说"发布我的技能"但没说哪个 → 扫描工作目录下含 SKILL.md 的子目录，列出可用 Skill 让用户选择
    - **多 Skill**：用户指定父目录，但其下有多个 Skill 子目录 → 列出所有 Skill，让用户选择一个或确认全部发布
    - **路径模糊**：用户说"发布 wx-peitu"但没给完整路径 → 在工作目录下搜索匹配的子目录，找到 1 个直接用，找到多个让用户选择，找到 0 个报错
20. **SkillHub 发布前 TRACE 五维度预检**（v5.3 新增，核心规则）：发布到 SkillHub 前必须对目标 Skill 执行 TRACE 五维度自检，任何维度 FAIL = 中止 SkillHub 发布并报告问题。GitHub 和 ClawHub 不受此限制（这两个平台无 TRACE 检测）：
    - **T（Trust 信任）**：安全红线扫描（无 curl/wget/eval/凭证硬编码）+ frontmatter 有 allowed-tools 声明（可选）+ 国内可用性
    - **R（Reliability 可靠）**：前置条件校验（规则17）+ 质量门禁（规则18）+ 边界输入处理（规则19）+ 异常处理反馈
    - **A（Applicability 适用）**：触发测试 — description 含核心触发词 + 有 Do NOT 排除范围
    - **C（Compliance 规范）**：Schema 检查 — 4 模块齐全（任务/输出格式/规则/示例）+ SKILL.md ≤200 行 + 示例含边界情况 + 规则通过实习生测试
    - **E（Effectiveness 有效）**：增量价值 — Skill 相比手动操作有明显增益（如自动化安全审查、版本号查重等）

## 执行流程

**读取 `references/publishing-guide.md` 获取完整发布流程。** 以下为摘要。

### Step 0: 前置条件校验（v5.2 新增）
执行规则17的4项前置条件校验（目录存在/SKILL.md存在/平台登录态/Git配置）+ 规则18的Skill质量门禁。任何一项不满足 = 中止发布，明确告知用户缺什么、怎么修。全部通过才进入 Step 1。

### Step 1: 仓库结构生成
生成标准目录：SKILL.md / README.md(中英双语) / CHANGELOG.md / LICENSE(MIT-0) / .gitignore / .claude-plugin/plugin.json。确认作者名、GitHub owner、版本号、ClawHub slug、SkillHub slug。SKILL.md frontmatter 必须同时包含 ClawHub 字段（name/description）和 SkillHub 字段（slug/displayName/version/summary/license）。

### Step 2: 安全审查
三类扫描（凭证/路径/危险命令），全部 PASS 才能继续。凭证扫描必须覆盖 `skh_` 前缀（SkillHub token）。分发物三维判定 + ClawHub slug 检查 + ClawHub 自动文件排除 + SkillHub slug 全网唯一性检查。

### Step 3: 版本号查重
- ClawHub：`clawhub inspect <slug>` 查看已发布版本列表，确认待发布版本号不重复。重复则递增 PATCH 后重新确认。
- SkillHub：版本号在 frontmatter 的 `version` 字段中，更新时保持 slug 不变，递增 version。
- GitHub：检查 git tag 是否已存在。

### Step 4: GitHub 推送
优先 git push → gh CLI → REST API 降级。创建 Release。

### Step 5: ClawHub 发布
`clawhub publish <path> --slug <slug> --version <version> --tags "<ASCII-only>" --changelog "<text>"`

### Step 6: SkillHub 发布（v5.1 新增，v5.3 加入 TRACE 预检）
```bash
# 1. 确认登录态
skillhub auth whoami

# 2. TRACE 五维度预检（v5.3 新增，规则20）
#    T: 安全红线扫描 + allowed-tools + 国内可用性
#    R: 前置条件 + 质量门禁 + 边界输入 + 异常处理
#    A: 触发测试（正例 + 反例）
#    C: Schema（4模块/200行/示例/实习生测试）
#    E: 增量价值
#    任何维度 FAIL = 中止 SkillHub 发布，报告问题

# 3. 临时移除不支持的文件类型（.gitignore/LICENSE/.claude-plugin/.github）
#    备份到临时目录，发布后立即恢复

# 4. dry-run 预检（必须通过）
skillhub publish <path> --dry-run

# 5. 正式发布
skillhub publish <path> --changelog "变更说明"

# 6. 立即恢复被移除的文件
```

> **Windows 注意**：`skillhub` 命令可能因 python3 stub 失效（exit code 9009），需用 `python "%USERPROFILE%\.skillhub\skills_store_cli.py"` 替代。详见 `references/skillhub-publishing.md`。
> **文件类型限制**：SkillHub 拒绝 `.gitignore`、`LICENSE`、`.claude-plugin/`、`.github/`，发布前必须临时移除，发布后立即恢复。
> **TRACE 预检**：SkillHub 平台会对上架技能执行 TRACE 五维度检测，本技能在发布前预执行同样的检测，避免上架后被扣分。

### Step 7: 发布后验证
GitHub 文件列表检查 + `clawhub inspect <slug>` 确认 + SkillHub 状态检查。检查 ClawHub Short summary 是否与 frontmatter description 一致，不一致则递增版本号重新发布。

## 示例

### 示例1：常见输入（完整 Skill 目录发布）

**用户输入**："帮我把 wx-peitu 技能发布到三平台，版本号 7.1.0"

**前置条件校验**：
- ✅ 目录 `d:\TRAE SOLO CN\project\wx-peitu` 存在且非空
- ✅ SKILL.md 存在
- ✅ ClawHub 已登录（clawhub whoami 通过）
- ✅ SkillHub 已登录（skillhub auth whoami 通过）
- ✅ Git 配置完整

**质量门禁**：
- ✅ SKILL.md 180行（<300）
- ✅ description 存在且含触发词
- ✅ description 含 Do NOT 范围声明

**安全审查结果**：
| 审查项 | 状态 | 详情 |
|--------|------|------|
| 凭证泄露 | PASS | 无 token/api_key/secret 硬编码 |
| 本地路径 | PASS | 无 C:\ 或 D:\ 绝对路径 |
| 危险命令 | PASS | 无 curl/wget/eval |
| 分发物判定 | PASS | 无 __pycache__/.clawhub/skill-card.md |

**版本号查重结果**：
| ClawHub 已发布版本 | 待发布版本 | 状态 |
|-------------------|-----------|------|
| v7.0.0 | v7.1.0 | 可发布 |

**发布结果**：
| 平台 | 地址 | 版本 | 状态 |
|------|------|------|------|
| GitHub | github.com/EdwardWason/wx-peitu | v7.1.0 | 成功 |
| ClawHub | clawhub.ai/skills/wx-peitu | v7.1.0 | 成功 |
| SkillHub | skillhub.cn/skills/wx-peitu | v7.1.0 | 成功 |

### 示例2：边界输入（目录不存在）

**用户输入**："发布 my-new-skill"

**前置条件校验失败**：
```
❌ 目录不存在或为空：d:\TRAE SOLO CN\project\my-new-skill
请确认 Skill 路径，或使用完整路径指定。
```

**中止发布，不继续执行后续步骤。**

### 示例3：边界输入（SKILL.md 缺失）

**用户输入**："发布 ./my-project"

**前置条件校验**：
- ✅ 目录存在且非空
- ❌ 未找到 SKILL.md，这不是一个有效的 Skill 目录

**中止发布，提示用户**：
```
❌ 目录 ./my-project 下未找到 SKILL.md。
这不是一个有效的 Skill 目录。
请确认目录路径，或先使用 skill-forge 创建 Skill。
```

### 示例4：边界输入（Skill 质量不达标）

**用户输入**："发布 ./my-skill，版本 1.0.0"

**前置条件校验**：✅ 全部通过

**质量门禁失败**：
```
❌ SKILL.md 过长（350行），建议精简到 200 行以内再发布。
❌ description 缺少 Do NOT 范围声明，可能导致误触发。
请修复以上问题后再发布。可调用 skill-forge 进行评估和优化。
```

**中止发布，建议先修复。**

### 示例5：异常输入（未登录 ClawHub）

**用户输入**："发布 ./wx-peitu"

**前置条件校验失败**：
```
❌ ClawHub 未登录，请先执行：
   clawhub login --token <your-token> --no-browser
```

**中止发布，不继续执行后续步骤。**

### 示例6：复杂输入（未指明哪个 Skill）

**用户输入**："发布我的技能"

**扫描工作目录**：
```
找到以下 Skill 目录：
1. d:\TRAE SOLO CN\project\wx-peitu (含 SKILL.md)
2. d:\TRAE SOLO CN\project\pic-book (含 SKILL.md)
3. d:\TRAE SOLO CN\project\web-to-fim (含 SKILL.md)

请指定要发布的 Skill 名称或序号，或回复"全部"发布所有 Skill。
```

**等待用户选择后继续。**

### 示例7：复杂输入（路径模糊）

**用户输入**："发布 wx-peitu"

**搜索匹配目录**：
```
找到 1 个匹配目录：d:\TRAE SOLO CN\project\wx-peitu
自动使用此目录继续发布。
```

### 示例8：SkillHub TRACE 预检失败

**用户输入**："发布 ./my-skill 到 SkillHub"

**TRACE 预检结果**：
```
T (Trust):       ✅ PASS
R (Reliability): ✅ PASS
A (Applicability): ❌ FAIL — description 缺少 Do NOT 范围声明
C (Compliance):  ❌ FAIL — 缺少"示例"模块
E (Effectiveness): ✅ PASS

TRACE 预检未通过（A/C 失败），中止 SkillHub 发布。
建议：先补全 Do NOT 声明和示例模块，再重新发布。
```

**中止 SkillHub 发布，GitHub 和 ClawHub 已发布的保留。**

## References

- **[`references/publishing-guide.md`](references/publishing-guide.md)** — 完整发布流程。仓库结构模板、安全审查、版本号查重、GitHub API降级、ClawHub CLI、SkillHub CLI、PowerShell兼容、故障排查。
- **[`references/skillhub-publishing.md`](references/skillhub-publishing.md)** — SkillHub 发布详细流程。CLI 安装、登录、frontmatter 兼容、dry-run 预检、正式发布、Windows 兼容、故障排查。
- **[`references/security-audit.md`](references/security-audit.md)** — 三层安全扫描（含扩展凭证模式 + SKILLHUB_TOKEN）+ 分发物判定 + ClawHub 自动文件排除 + 修复指南。
- **[`references/publish-procedures.md`](references/publish-procedures.md)** — 推送降级链 + gh CLI + Release + ClawHub + SkillHub + 版本号查重 + 故障排查。
- **[`references/change-detection.md`](references/change-detection.md)** — 变更检测 + 版本 bump + Conventional Commits。
- **[`references/changelog-generation.md`](references/changelog-generation.md)** — git log 提取 + CHANGELOG 生成 + Release Notes 转换。
- **[`references/repo-structure.md`](references/repo-structure.md)** — 仓库结构模板 + README 21 章节 + 智能适配 + .gitignore 模板 + frontmatter 兼容模板。
