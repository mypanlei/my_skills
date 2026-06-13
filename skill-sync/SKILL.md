---
name: skill-sync
description: "Sync skills from my_skills (source of truth) to category repositories. Uses NTFS junctions (mklink /J) for zero-copy local access. Scan, classify, link, commit, and push. Use when: adding a new skill, updating an existing skill, wanting to distribute skills to their target repos, syncing everything to GitHub, or setting up skills after cloning a repo. Chinese: 技能同步、分发skill、备份skill、同步到分类仓库、clone后skill重新链接。"
---

# Skill Sync — Multi-Repo Skill Distributor

扫描 `my_skills`（唯一真源），按分类规则自动分发到各分类仓库，使用 NTFS junction 实现零拷贝本地访问，并推送配置至 GitHub。

## 架构

```
my_skills/                  ← 唯一真源 (source of truth)
├── elite-powerpoint-designer/
├── gitmemo/
├── requirement-validation/
├── tailored-resume-generator/
├── resume-formatter/
└── skill-sync/             ← 本 skill
         │
         │  mklink /J (NTFS junction — 零拷贝)
         ▼
┌─────────────────┬──────────────────┬─────────────────┐
│  ai_knowledge   │     resume       │      mleco      │
│ .agents/skills/ │ .agents/skills/  │ .github/skills/ │
│                 │                  │                 │
│ .symlinks  ←── 配置文件，记录每个 skill 的链接目标       │
│ .gitignore ←── 忽略 junction 目录，防止 git 追踪实体文件   │
│                 │                  │                 │
│ • powerpoint ◀──┼── NTFS junction ──┼──→  my_skills   │
│ • gitmemo   ◀──┼── NTFS junction ──┼──→  my_skills   │
│ • resume-*  ◀──┼── NTFS junction ──┼──→  my_skills   │
│ • req-validate ◀┼── NTFS junction ──┼──→  my_skills   │
└─────────────────┴──────────────────┴─────────────────┘
         │                │                  │
         ▼                ▼                  ▼
     GitHub           GitHub             GitHub
  (存储 .symlinks   (存储 .symlinks   (存储 .symlinks
   配置文件)         配置文件)          配置文件)
```

## 核心设计原则

- **真源唯一**：`my_skills` 是唯一的 skill 编辑源。目标仓库中不再存储 skill 实体文件。
- **零拷贝访问**：通过 NTFS junction (`mklink /J`) 链接，Windows 直接跟随重解析点访问原文件。
- **Git 友好**：Git 仓库仅追踪 `.symlinks` 配置文件 + `.gitignore`，不追踪 junction 目录内的实体文件。
- **Clone 后可重建**：在其他机器 clone 后，运行 `skill-sync setup` 即可重建所有 junction。

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

### 模式 A: 首次同步 / 更新 .symlinks 配置

当 `my_skills` 中新增、删除或重新分类 skill 时使用。

#### Step 1: 扫描 my_skills

```bash
ls -d /c/Users/mypan/OneDrive/workspace/my_skills/*/
```

识别每个包含 `SKILL.md` 的子目录为一个 Skill。

#### Step 2: 读取 SKILL.md 元数据

```bash
head -6 <skill>/SKILL.md
```

提取 `name` 和 `description` 字段。

#### Step 3: 自动分类

根据关键字匹配规则，将每个 skill 映射到目标仓库列表。

示例：
- `elite-powerpoint-designer` → `ai_knowledge` (powerpoint, presentation)
- `gitmemo` → `ai_knowledge` (memory, agent, gitmemo)
- `tailored-resume-generator` → `resume` (resume, job, career)
- `resume-formatter` → `resume` (resume, ATS)
- `requirement-validation` → `mleco`, `bdd` (requirement, validation, audit)

#### Step 4: 更新 .symlinks 配置

对于每个目标仓库，更新 `.symlinks` 文件：

```bash
# 格式: <skill-name> <relative-target-path>
# ai_knowledge/.agents/skills/.symlinks
elite-powerpoint-designer ../../../my_skills/elite-powerpoint-designer
gitmemo ../../../my_skills/gitmemo

# resume/.agents/skills/.symlinks
resume-formatter ../../../my_skills/resume-formatter
tailored-resume-generator ../../../my_skills/tailored-resume-generator

# mleco/.github/skills/.symlinks
requirement-validation ../../../my_skills/requirement-validation
```

#### Step 5: 更新 .gitignore

确保每个 skills 目录有 `.gitignore` 忽略 junction 路径：

```bash
# .gitignore
# Junction symlinks managed by skill-sync
elite-powerpoint-designer/
gitmemo/
```

#### Step 6: 创建 Junction（本地）

为每个 (skill, target_repo) 组合创建 NTFS junction：

```bash
# 如果目标已存在（旧实体目录或旧 junction），先删除
rm -rf <target-path>

# 创建 junction (Windows mklink /J)
cmd //c "mklink /J <target-path> <my_skills-source-path>"
```

**注意**：`bdd` 不是 git 仓库，仅做本地 junction 创建。

#### Step 7: 处理 Git 锁文件

如果在 Windows 下遇到 `.git/index.lock` 僵死锁：

```bash
# 杀掉所有 git 进程
powershell -NoProfile -Command "Get-Process git -ErrorAction SilentlyContinue | Stop-Process -Force"

# 删除僵死锁
rm -f <repo>/.git/index.lock
```

#### Step 8: 提交 & 推送

对于每个有变更的仓库：

```bash
# 1. 先拉取远程最新
git -C <repo> pull --ff-only

# 2. 添加变更 (.symlinks + .gitignore + 删除的旧实体文件)
git -C <repo> add .

# 3. 提交
git -C <repo> commit -m "sync: update skill symlink config from my_skills

- Replace skill entity files with .symlinks config
- Add .gitignore to exclude junction directories
- Skills now accessed via NTFS junctions to my_skills

Co-Authored-By: Claude <noreply@anthropic.com>"

# 4. 推送
git -C <repo> push
```

### 模式 B: Clone 后恢复 Junction（setup）

在其他机器 clone 目标仓库后，skill 实体文件不存在，需要从 `.symlinks` 重建 junction：

```bash
# 对每个目标仓库执行：
WORKSPACE="/c/Users/mypan/OneDrive/workspace"

# 读取 .symlinks 文件
while read -r skill target; do
  # 跳过注释和空行
  [[ "$skill" =~ ^#.*$ || -z "$skill" ]] && continue
  
  # 删除旧目录/junction
  rm -rf "$REPO/<skills_path>/$skill"
  
  # 重建 junction
  cmd //c "mklink /J $REPO/<skills_path>/$skill $WORKSPACE/$target"
done < "$REPO/<skills_path>/.symlinks"
```

### 模式 C: 日常验证

快速检查所有 junction 是否正常工作：

```bash
# 扫描所有 skill 及其 junction 状态
for skill in /c/Users/mypan/OneDrive/workspace/my_skills/*/; do
  name=$(basename "$skill")
  [ "$name" = "skill-sync" ] && continue
  echo "=== $name ==="
  for repo in ai_knowledge resume mleco bdd; do
    for skills_dir in .agents/skills .github/skills; do
      target="/c/Users/mypan/OneDrive/workspace/$repo/$skills_dir/$name"
      if [ -d "$target" ]; then
        # Check if it's a junction
        head -1 "$target/SKILL.md" > /dev/null 2>&1 && echo "  → $repo ($skills_dir) ✅" || echo "  → $repo ($skills_dir) ❌ broken"
      fi
    done
  done
done
```

## .symlinks 文件格式

```
# Symlink mappings for skills sourced from my_skills
# Format: <skill-name> <relative-target-path>
# Generated by skill-sync. Do not edit manually.
<skill-name-1> <relative-path-from-skills-dir-to-my_skills>
<skill-name-2> <relative-path-from-skills-dir-to-my_skills>
```

示例 (`ai_knowledge/.agents/skills/.symlinks`):
```
# Symlink mappings for skills sourced from my_skills
# Format: <skill-name> <relative-target-path>
# Generated by skill-sync. Do not edit manually.
elite-powerpoint-designer ../../../my_skills/elite-powerpoint-designer
gitmemo ../../../my_skills/gitmemo
```

## 边界情况处理

| 情况 | 处理方式 |
|------|---------|
| `.symlinks` 中引用的 skill 不存在 | 跳过，输出警告 |
| Skill 已存在于目标且为 junction | 检查 junction 目标是否正确 → 更新或跳过 |
| Skill 已存在于目标但为实体目录 | 删除实体目录 → 创建 junction |
| `.git/index.lock` 僵死 | kill git 进程 → 删除锁文件 → 重试 |
| `pull` 冲突（untracked overwritten） | 备份冲突文件 → 删除 → pull → 对比 |
| 目标仓库无 remote | 仅本地 junction，跳过 git push |
| 目标 skills 路径不存在 | 自动创建 `.agents/skills/` 或 `.github/skills/` |
| 非 git 仓库（如 bdd） | 仅创建 junction，跳过所有 git 操作 |
| Clone 后 my_skills 不在预期位置 | 编辑 `.symlinks` 文件调整相对路径 |

## 分类仓库完整列表

| 仓库 | 本地路径 | GitHub | Skills 路径 | Git? |
|------|---------|--------|------------|------|
| `ai_knowledge` | `~/OneDrive/workspace/ai_knowledge` | `mypanlei/ai_knowledge` | `.agents/skills/` | ✅ |
| `resume` | `~/OneDrive/workspace/resume` | `mypanlei/resume` | `.agents/skills/` | ✅ |
| `mleco` | `~/OneDrive/workspace/mleco` | `mypanlei/mleco` | `.github/skills/` | ✅ |
| `study_pool` | `~/OneDrive/workspace/study_pool` | `mypanlei/study_pool` | `.claude/skills/` | ✅ |
| `bdd` | `~/OneDrive/workspace/bdd` | 无 | `.github/skills/` | ❌ |

## 注意事项

- **真源唯一**：`my_skills` 是唯一的 skill 编辑源。不要在目标仓库中直接编辑 skill。
- **Junction 非 Symlink**：Windows NTFS junction 不同于 POSIX symlink。Junction 仅支持目录、仅支持本地卷。Git 默认看穿 junction（`core.symlinks=false`），所以必须用 `.gitignore` 排除 junction 路径。
- **相对路径**：`.symlinks` 中的路径是相对于 skills 目录的相对路径，确保仓库可移植。
- **双向保护**：如果目标仓库有额外 skill 不在 `my_skills` 中（如 `resume/resume-formatter` 之前是独立维护的），不要删除它们——它们可能是该仓库独有的。`.symlinks` 只管理来自 `my_skills` 的 skill。
- **提交信息**：始终以 `sync: update skill symlink config from my_skills` 作为提交标题。
- **Co-Authored-By**：使用 `Co-Authored-By: Claude <noreply@anthropic.com>` 标记 AI 辅助提交。
