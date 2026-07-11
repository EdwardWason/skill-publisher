---
name: "skill-publisher"
description: "技能发布 — 将已有 Skill 推送到 GitHub + ClawHub。当用户说 技能发布/发布技能/更新技能/迭代技能 时触发。含安全审查、隐私清洗、版本号查重、仓库结构生成、ClawHub 自动文件排除。Do NOT use for creating skill content, general coding, or non-skill projects."
---

# 技能发布

将已有 Skill 推送到 GitHub + ClawHub，含安全审查、隐私清洗、版本号查重、标准仓库结构生成、ClawHub 自动文件排除。

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

## 执行流程

**读取 `references/publishing-guide.md` 获取完整发布流程。** 以下为摘要。

### Step 1: 仓库结构生成
生成标准目录：SKILL.md / README.md(中英双语) / CHANGELOG.md / LICENSE(MIT-0) / .gitignore / .claude-plugin/plugin.json。确认作者名、GitHub owner、版本号、ClawHub slug。

### Step 2: 安全审查
三类扫描（凭证/路径/危险命令），全部 PASS 才能继续。分发物三维判定 + ClawHub slug 检查 + ClawHub 自动文件排除。

### Step 3: 版本号查重（v5.0 新增）
```bash
clawhub inspect <slug>
```
查看已发布版本列表，确认待发布版本号不重复。重复则递增 PATCH 后重新确认。

### Step 4: GitHub 推送
优先 git push → gh CLI → REST API 降级。创建 Release。

### Step 5: ClawHub 发布
`clawhub publish <path> --slug <slug> --version <version> --tags "<ASCII-only>" --changelog "<text>"`

### Step 6: 发布后验证
GitHub 文件列表检查 + `clawhub inspect <slug>` 确认。检查 Short summary 是否与 frontmatter description 一致，不一致则递增版本号重新发布。

## References

- **[`references/publishing-guide.md`](references/publishing-guide.md)** — 完整发布流程。仓库结构模板、安全审查、版本号查重、GitHub API降级、ClawHub CLI、PowerShell兼容、故障排查。
- **[`references/security-audit.md`](references/security-audit.md)** — 三层安全扫描（含扩展凭证模式）+ 分发物判定 + ClawHub 自动文件排除 + 修复指南。
- **[`references/publish-procedures.md`](references/publish-procedures.md)** — 推送降级链 + gh CLI + Release + ClawHub + 版本号查重 + 故障排查。
- **[`references/change-detection.md`](references/change-detection.md)** — 变更检测 + 版本 bump + Conventional Commits。
- **[`references/changelog-generation.md`](references/changelog-generation.md)** — git log 提取 + CHANGELOG 生成 + Release Notes 转换。
- **[`references/repo-structure.md`](references/repo-structure.md)** — 仓库结构模板 + README 21 章节 + 智能适配 + .gitignore 模板。
