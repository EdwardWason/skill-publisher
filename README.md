# Skill Publisher · 技能一键发布与迭代工具

🌍 English version: [README.en.md](README.en.md)

[![Stars](https://img.shields.io/github/stars/EdwardWason/skill-publisher?style=flat-square)](https://github.com/EdwardWason/skill-publisher)
[![许可证](https://img.shields.io/badge/license-MIT--0-green?style=flat-square)](LICENSE)
[![ClawHub](https://img.shields.io/badge/ClawHub-skill--publisher--ai-orange?style=flat-square)](https://clawhub.ai/skills/skill-publisher-ai)
[![Skill](https://img.shields.io/badge/Claude%20Code-Skill-blue?style=flat-square)](SKILL.md)

[装上就能用](#30-秒开始) · [核心机制](#核心机制) · [已知限制](#已知限制) · [License](#license)

---

_「从本地 Skill 到三平台上线，安全审查先行，每一步都向你确认。」_

---

**Skill Publisher** 不是手动 git push 的备忘录，是从 Skill 开发完成到线上发布的全流程自动化工具。它同时支持**新建发布**和**迭代更新**两种工作流，覆盖仓库结构生成、安全审查、变更检测、自动版本 bump、CHANGELOG 生成、推送降级、Release 创建、ClawHub 发布的完整链路。

---

> ⚠️ **重要提示**：本工具会修改 Git 仓库、创建 GitHub Release、发布到 ClawHub/SkillHub 外部平台。这些操作对外可见且可能不可逆，请确认后再执行。

## 效果

- 🏗 **双工作流**：新建发布（工作流 A）+ 迭代更新（工作流 B），自动检测模式
- 📐 **产品着陆页 README**：21 章节标准，金句开篇 → 否定对比 → 效果展示 → 已知限制，从 SKILL.md 自动提取信息
- 🌍 **双语独立文件**：中文 `README.md` + 英文 `README.en.md`，顶部互链切换
- 🔒 **三层安全扫描（v5.0 扩展）**：凭证泄露（含 IMA/飞书/SkillHub 新模式）+ 本地路径 + 危险命令，任何 FAIL 阻止发布
- 🔍 **版本号查重（v5.0 新增）**：ClawHub 发布前自动查重版本号，重复则递增 PATCH，避免 "Version already exists" 错误
- 🚫 **ClawHub 自动文件排除（v5.0 新增）**：自动识别并排除 `skill-card.md`、`.clawhub/`、`_meta.json` 等 ClawHub 自动生成文件
- 🧪 **SkillHub dry-run 预检（v5.1 新增）**：SkillHub 发布前自动 `--dry-run` 预检，检查 frontmatter 格式，通过后才正式发布
- 🚀 **三平台同步发布（v5.1 新增）**：GitHub + ClawHub + SkillHub 三平台一键同步，frontmatter 兼容双平台字段
- 📊 **变更检测 + 自动版本 bump**：按文件类别映射版本影响，决策树推荐 MAJOR/MINOR/PATCH
- 📝 **自动 CHANGELOG 生成**：从 git log 提取，Conventional Commits 分类，空类别不输出
- 🔄 **推送降级链**：git push → gh CLI → REST API，网络不稳定也能发布
- 📋 **社区模板**：bug_report / feature_request / question / config / pull_request_template
- 🏷 **来源识别**：SKILL.md provenance 块，确保 Skill 产出物可溯源

## 适合 / 不适合

✅ **适合**
- Skill 开发完成后一键发布到 GitHub + ClawHub
- 已有 Skill 需要迭代更新（自动检测变更、bump 版本、生成 CHANGELOG）
- 想要标准化仓库结构（README、社区模板、provenance）
- 需要安全审查防止凭证泄露

❌ **不适合**
- Skill 内容创建（用对应的 Skill 本身）
- 非 Skill 项目的发布
- 代码开发

## 30 秒开始

```bash
# ClawHub 安装（推荐）
clawhub install skill-publisher-ai

# 或手动安装
git clone https://github.com/EdwardWason/skill-publisher.git
```

安装后对 Agent 说（需明确表达发布到外部平台的意图）：

```
把 wx-peitu 发布到 GitHub 和 ClawHub
```

```
发布技能更新 react-design-draft
```

```
迭代技能发布 skill-publisher
```

## 常见使用场景

> ⚠️ **前置条件**：以下触发词仅在用户明确要求发布到外部平台时生效。单纯的"更新技能"（指修改内容）不触发本技能。

| 场景 | 工作流 | 触发词（需带发布意图） |
|------|--------|--------|
| 新 Skill 开发完成，首次发布 | 工作流 A | "技能发布到三平台"/"发布技能到 GitHub" |
| 已有 Skill 增加了新功能 | 工作流 B | "发布技能更新"/"迭代技能发布" |
| 修复了 Skill 的 bug | 工作流 B | "发布技能更新，修了个 bug" |
| 重写了 Skill 的设计系统 | 工作流 B | "迭代技能发布，破坏性变更" |

## 核心机制

### 双工作流自动检测

> ⚠️ 以下触发词仅在用户明确要求发布到外部平台时生效，单纯的"更新技能"不触发。

```
用户说"技能发布到三平台"/"发布技能到 GitHub"?
  → 工作流 A: 仓库结构生成 → 安全审查 → 推送 → 发布

用户说"发布技能更新"/"迭代技能发布"?
  → 工作流 B: 变更检测 → 版本 bump → CHANGELOG → 安全审查 → 推送 → 发布
```

### 工作流 A：新建发布（15 步，v5.1 新增 SkillHub 发布）

```
生成目录结构 → 生成 README(中/英) → 生成辅助文件 → 添加 provenance → 确认信息
 安全扫描 → 分发物判定 → ClawHub 自动文件排除 → slug 检查 → 版本号查重
 → 推送 → 创建 Release → ClawHub 发布 → SkillHub dry-run → SkillHub 发布 → 验证
```

### 工作流 B：迭代更新（12 步）

```
检测变更 → 分类文件 → 推荐 bump → 确认版本 → 三文件同步
→ 提取 git log → 解析 commits → 生成 CHANGELOG → 安全审查
→ 推送 → 创建/更新 Release → ClawHub 更新 → 验证
```

### 变更检测 → 版本 bump 决策树

| 变更类型 | 版本影响 | 示例 |
|---------|---------|------|
| SKILL.md frontmatter 变更 | MAJOR | 改了 name/description |
| 新增/删除规则 | MINOR | 加了 2 条规则 |
| 新增 references 文件 | MINOR | 新增 workflow.md |
| references 内容重写 | MINOR | design-system.md 大改 |
| README 文档更新 | PATCH | 新增 FAQ 章节 |
| 社区模板新增 | PATCH | 添加 config.yml |

### 推送降级链

```
git push（优先，最快）
  → 失败 → gh CLI（次优，自动认证）
  → 失败 → REST API 逐文件上传（最后手段）
```

### README 产品着陆页 21 章节标准

按 Skill 复杂度智能适配：简单 Skill（≤5 条规则）只生成必选章节，复杂 Skill 生成全部 21 章节。

## 目录结构

```
skill-publisher/
├── SKILL.md                              # 双工作流定义（25 步 + 15 条规则）
├── README.md                             # 中文文档
├── README.en.md                          # English docs
├── CHANGELOG.md                          # 版本记录
├── LICENSE                               # MIT-0
├── .claude-plugin/
│   └── plugin.json                       # Claude Code 元数据
├── .github/                              # 社区模板
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml
│   │   ├── feature_request.yml
│   │   ├── question.yml
│   │   └── config.yml
│   └── pull_request_template.md
└── references/                           # 参考文档
    ├── repo-structure.md                 # 仓库结构模板 + README 21 章节 + 智能适配
    ├── security-audit.md                 # 三层扫描 + 分发物判定 + 修复指南
    ├── publish-procedures.md             # 推送降级链 + gh CLI + Release + ClawHub
    ├── change-detection.md               # 变更检测 + 版本 bump + Conventional Commits
    └── changelog-generation.md           # git log 提取 + CHANGELOG 生成 + Release Notes 转换
```

## 已知限制

- **仅支持 Skill 项目**：不做非 Skill 项目的发布（无 package.json 管理、无 CI/CD 配置）
- **Windows 优先**：PowerShell 兼容性优先处理，macOS/Linux 部分命令需适配
- **git push 超时常见**：国内网络环境 git push 443 超时频繁，降级方案是核心能力
- **无 GUI**：完全依赖 Agent 平台（TRAE / Claude Code / Codex）
- **ClawHub slug 不可改**：一旦发布 slug 不能修改，只能发新版本
- **ClawHub Short summary 需重新发布**：更新 frontmatter description 后必须递增版本号重新发布，否则 Short summary 不更新（v5.0 新增）

## 核心设计原则

1. **安全第一**：任何 FAIL 阻止发布，宁可多扫一次不可漏放一个凭证
2. **降级优先**：网络不稳定是常态，三级降级链确保总能发布成功
3. **自动化但不盲目**：版本 bump 必须向用户确认，CHANGELOG 自动生成但可编辑
4. **产品思维**：README 是产品着陆页不是技术规格书，让人想用比让人理解更重要
5. **Dogfooding**：skill-publisher 用自身流程发布自己

## FAQ

**支持已有仓库的更新吗？**
支持。说"更新 xxx"触发工作流 B，自动检测变更、推荐版本 bump、生成 CHANGELOG。

**git push 超时怎么办？**
自动降级：先尝试 gh CLI（自动认证），再降级为 REST API 逐文件上传。

**版本号怎么决定？**
按变更文件类别自动推荐：SKILL.md 规则变更 → MINOR，README 文档更新 → PATCH，frontmatter 破坏性变更 → MAJOR。推荐后需用户确认。

**ClawHub 报 "Version already exists" 怎么办？**
v5.0 新增版本号查重：发布前自动 `clawhub inspect <slug>` 查重，重复则递增 PATCH。

**ClawHub Short summary 没更新？**
ClawHub 用首次发布的 description 作为 Short summary。更新 frontmatter description 后必须递增版本号重新发布。

**SkillHub 发布报 409 slug 已被占用？**
slug 必须**全网唯一**。建议带 handle 后缀（如 `skill-publisher-ai`）。修改 SKILL.md frontmatter 中的 slug 后重试。

**SkillHub 发布报 exit code 9009（Windows）？**
Windows 上 `python3` 是 Store stub，需用 `python` 直接调用：
```powershell
python "%USERPROFILE%\.skillhub\skills_store_cli.py" publish <path> --changelog "xxx"
```

**SkillHub 发布报 403 请先完成实名认证？**
浏览器打开 skillhub.cn → 个人中心 → 实名认证 → 完成人脸核身。

**README 太长怎么办？**
智能适配：简单 Skill 只生成 10 个必选章节，复杂 Skill 生成全部 21 章节。

**Conventional Commits 是强制的吗？**
不是。如果 commit message 不符合规范，会从文件 diff 分析推断变更分类。

---

## License

MIT-0 © 2026