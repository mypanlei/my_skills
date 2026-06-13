---
name: skill-sync
description: "Sync skills from my_skills (source of truth) to category repositories. Scan, classify, copy, commit, and push. Use when: adding a new skill, updating an existing skill, wanting to distribute skills to their target repos, or syncing everything to GitHub. Chinese: 技能同步、分发skill、备份skill、同步到分类仓库."
---

# Skill Sync — Multi-Repo Skill Distributor

扫描 `my_skills`（唯一真源），按分类规则自动分发到各分类仓库，并推送至 GitHub。

## 架构

```
my_skills/                  ← 唯一真源 (source of truth)
├── elite-powerpoint-designer/
├── gitmemo/
├── requirement-validation/
├── tailored-resume-generator/
└── skill-sync/             ← 本 skill
         │
         │  分类 & 同步
         ▼
┌─────────────────┬──────────────────┬─────────────────┐
│  ai_knowledge   │     resume       │      mleco      │
│  .agents/skills │ .agents/skills   │ .github/skills  │
│                 │                  │                 │
│  • powerpoint   │  • resume-gen    │  • req-validate │
│  • gitmemo      │  • formatter     │                 │
└─────────────────┴──────────────────┴─────────────────┘
         │                │                  │
         ▼                ▼                  ▼
     GitHub           GitHub             GitHub
```

## 分类规则

根据 Skill 的 `name`、`description` 和功能领域自动判断归属。一个 skill 可归属到多个仓库。

| 分类仓库 | Skills 路径 | GitHub Remote | 匹配规则 |
|----------|------------|---------------|---------|
| `ai_knowledge` | `.agents/skills/` | `mypanlei/ai_knowledge` | AI/agent 工具、知识管理、powerpoint、memory、通用效率 |
| `resume` | `.agents/skills/` | `mypanlei/resume` | resume/CV/求职/面试/职业发展/简历 |
| `mleco` | `.github/skills/` | `mypanlei/mleco` | requirement/需求/验证/ML/工程流程/BDD |
| `bdd` | `.github/skills/` | *无 remote* | requirement/需求/BDD/测试 (仅本地) |

### 分类关键字

```
ai_knowledge:  powerpoint, ppt, presentation, slide, memory, agent,
               AI, knowledge, tool, general, productivity, gitmemo

resume:        resume, CV, job, career, interview, ATS, 简历, 求职

mleco:         requirement, validation, review, spec, audit,
               ML, pipeline, process, compliance, 需求, 验证
```

## 工作流

### Step 1: 扫描 my_skills

```bash
ls -d /c/Users/mypan/OneDrive/workspace/my_skills/*/
```

识别每个包含 `SKILL.md` 的子目录为一个 Skill。

### Step 2: 读取 SKILL.md 元数据

```bash
head -6 <skill>/SKILL.md
```

提取 `name` 和 `description` 字段。

### Step 3: 自动分类

根据关键字匹配规则，将每个 skill 映射到目标仓库列表。

示例：
- `elite-powerpoint-designer` → `ai_knowledge` (powerpoint, presentation)
- `gitmemo` → `ai_knowledge` (memory, agent, gitmemo)
- `tailored-resume-generator` → `resume` (resume, job, career)
- `requirement-validation` → `mleco`, `bdd` (requirement, validation, audit)

### Step 4: 复制到目标仓库

对于每个 (skill, target_repo) 组合：

```bash
# 确定目标路径
# ai_knowledge, resume → .agents/skills/<skill-name>/
# mleco, bdd → .github/skills/<skill-name>/

# 先比较差异
diff -rq /path/to/my_skills/<skill> /path/to/<repo>/<skills_path>/<skill>

# 若有差异或目标不存在，则复制
cp -r /path/to/my_skills/<skill> /path/to/<repo>/<skills_path>/
```

**注意**：`bdd` 不是 git 仓库，仅做本地文件复制。`my_knowledge` 同样。

### Step 5: 检查 Git 状态

对于每个有 remote 的目标仓库：

```bash
git -C <repo> status
git -C <repo> remote -v
```

### Step 6: 处理 Git 锁文件

如果在 Windows 下遇到 `.git/index.lock` 僵死锁：

```bash
# 杀掉所有 git 进程
powershell -NoProfile -Command "Get-Process git -ErrorAction SilentlyContinue | Stop-Process -Force"

# 删除僵死锁
rm -f <repo>/.git/index.lock
```

### Step 7: 拉取 & 提交 & 推送

对于每个有变更的仓库：

```bash
# 1. 先拉取远程最新
git -C <repo> pull --ff-only

# 2. 如果有本地未跟踪文件冲突（untracked would be overwritten）
#    先备份 → 删除本地文件 → pull → 再对比还原

# 3. 添加变更
git -C <repo> add .

# 4. 提交
git -C <repo> commit -m "sync: update skills from my_skills

Co-Authored-By: Claude <noreply@anthropic.com>"

# 5. 推送
git -C <repo> push
```

### Step 8: 输出同步报告

汇总输出：

```
Skill Sync Report
=================
Source: my_skills (4 skills)

elite-powerpoint-designer → ai_knowledge ✅ synced
gitmemo → ai_knowledge ✅ synced
requirement-validation → mleco ✅ synced
requirement-validation → bdd ✅ synced (local only)
tailored-resume-generator → resume ✅ synced

GitHub push:
  ai_knowledge ✅ pushed
  resume ✅ pushed
  mleco ⚠️ needs manual intervention
```

## 快捷脚本

也可以用下面的单行扫描快速了解状态：

```bash
# 扫描所有 skill 及其目标仓库
for skill in /c/Users/mypan/OneDrive/workspace/my_skills/*/; do
  name=$(basename "$skill")
  [ "$name" = "skill-sync" ] && continue
  echo "=== $name ==="
  for repo in ai_knowledge resume mleco bdd; do
    for skills_dir in .agents/skills .github/skills; do
      target="/c/Users/mypan/OneDrive/workspace/$repo/$skills_dir/$name"
      [ -d "$target" ] && echo "  → $repo ($skills_dir)"
    done
  done
done
```

## 边界情况处理

| 情况 | 处理方式 |
|------|---------|
| Skill 已存在于目标且内容相同 | 跳过，不产生空提交 |
| Skill 已存在于目标但内容不同 | 覆盖为目标源 (`my_skills`) 的版本 |
| `.git/index.lock` 僵死 | kill git 进程 → 删除锁文件 → 重试 |
| `pull` 冲突（untracked overwritten） | 备份冲突文件 → 删除 → pull → 对比 |
| 目标仓库无 remote | 仅本地复制，跳过 git push |
| 目标 skills 路径不存在 | 自动创建 `.agents/skills/` 或 `.github/skills/` |
| 非 git 仓库（如 bdd） | 仅 `cp -r`，跳过所有 git 操作 |

## 分类仓库完整列表

| 仓库 | 本地路径 | GitHub | Skills 路径 | Git? |
|------|---------|--------|------------|------|
| `ai_knowledge` | `~/OneDrive/workspace/ai_knowledge` | `mypanlei/ai_knowledge` | `.agents/skills/` | ✅ |
| `resume` | `~/OneDrive/workspace/resume` | `mypanlei/resume` | `.agents/skills/` | ✅ |
| `mleco` | `~/OneDrive/workspace/mleco` | `mypanlei/mleco` | `.github/skills/` | ✅ |
| `study_pool` | `~/OneDrive/workspace/study_pool` | `mypanlei/study_pool` | `.claude/skills/` | ✅ |
| `bdd` | `~/OneDrive/workspace/bdd` | 无 | `.github/skills/` | ❌ |
| `my_knowledge` | `~/OneDrive/workspace/my_knowledge` | 无 | 无 | ❌ |

## 注意事项

- **真源唯一**：`my_skills` 是唯一的 skill 编辑源。不要在目标仓库中直接编辑 skill。
- **双向同步**：如果目标仓库有额外 skill 不在 `my_skills` 中（如 `resume/resume-formatter`），不要删除它们——它们可能是该仓库独有的。
- **提交信息**：始终以 `sync: update skills from my_skills` 作为提交标题，方便追溯。
- **Co-Authored-By**：使用 `Co-Authored-By: Claude <noreply@anthropic.com>` 标记 AI 辅助提交。
