# 个人开发约定

## 分支策略

| 分支 | 用途 | 是否同步上游 |
|------|------|--------------|
| `pleggen_workspace` | 私人实验田：`.claude/`、`docs/`、`CLAUDE.md`、个人配置、实验性改动 | ❌ 不同步 |
| `main` (本地) | 干净的主线，与上游 `OpenWebGAL:main` 同步 | ❌ **不作为 PR base** |
| `OpenWebGAL/main` | 上游稳定版（tag 发布）| `git fetch upstream` 拉更新 |
| `OpenWebGAL/dev` | **上游开发主线（PR base）** | `git fetch upstream` 拉更新 |
| `fix/<num>-<desc>` | 修复分支，**从 `upstream/dev` 拉取** | ✅ 提 PR 到 `OpenWebGAL/dev` |
| `feat/<desc>` | 新功能分支，**从 `upstream/dev` 拉取** | ✅ 提 PR 到 `OpenWebGAL/dev` |

### ⚠️ 重要：修复 / 新功能分支必须从 `dev` 拉，不要从 `main` 拉

**原因**：
- 上游 `dev` 是开发主线，新 commit 持续合入
- 上游 `main` 是稳定版（只在新版本发布时合入）
- 从 `main` 拉分支 → 你的 PR 会基于"旧 base"，`merge-base` 计算后，PR 会多出一个上游 dev 的 merge commit
- 从 `dev` 拉分支 → PR 只包含你的 commit，干净

**示例**（PR #978 的教训）：

| 拉取方式 | 结果 |
|----------|------|
| ❌ 从 `main` 拉（`5a9e59ad`）| PR 多了 1 个 merge commit（`Merge pull request #970 from OpenWebGAL/dev`）|
| ✅ 从 `upstream/dev` 拉（`5f569eb5`）| PR 只有 3 个 commit，干净 |

## 同步上游更新

**首次配置 upstream**（如果还没有）：
```bash
git remote add upstream https://github.com/OpenWebGAL/WebGAL.git
```

**常规同步**：
```bash
# 同步 main
git checkout main
git fetch upstream
git merge upstream/main      # 或 git rebase upstream/main
git push origin main

# 同步 dev
git checkout main            # 切到 main
git fetch upstream
git merge upstream/dev       # 把上游 dev 的改动合到 main（dev > main）
git push origin main
```

**为什么把 dev 同步到 main**：
- 本地 `main` 同时跟踪 `upstream/main` 和 `upstream/dev`
- 这样拉修复分支时，base 永远最新

## 提 PR 流程

1. **不要**在 `pleggen_workspace` 上直接做 Bug 修复
2. 切到 `main` → `git fetch upstream` 同步 → 拉新分支（**从 `upstream/dev`**）
   ```bash
   git checkout main
   git fetch upstream
   git checkout -b fix/<issue-num>-<short-desc> upstream/dev
   ```
3. 修改、提交（提交粒度要小：`fix: 简述 (close #xxx)`）
4. 推送到 fork
   ```bash
   git push origin fix/<issue-num>-<short-desc>
   ```
5. GitHub 网页上 Compare & Pull Request
   - **Base**: `OpenWebGAL/WebGAL:dev`（**不是 main**）
   - **Head**: `CCreative-Od1L:fix/<issue-num>-<short-desc>`

### Why: 为什么不在 `pleggen_workspace` 上做 Bug 修复？

#### 场景对比

**❌ 在 `pleggen_workspace` 上做 Bug 修复：**

```
pleggen_workspace 提交历史（假设）：
1. chore: add CLAUDE.md              ← 私人配置
2. docs: add issues-screening.md     ← 私人笔记
3. chore: 调整 .gitignore            ← 私人配置
4. fix: 修复快进图标状态 (close #862) ← 想要提 PR
5. wip: 测试 Live2D 动画             ← 半成品实验
```

当你想提 PR #4 时：
- 选项 A：推整个分支 → **PR 里会夹带 1、2、3、5** → 维护者懵了
- 选项 B：只 cherry-pick #4 → 可以，但**步骤繁琐**，而且 #4 之后还有 #5，范围不好定
- 选项 C：先 rebase 把 #4 提到最前 → 容易出错，污染历史

**✅ 在 `main` 拉新分支做 Bug 修复：**

```
upstream/dev → 拉出 fix/862-icon-reset
  └─ fix: 修复快进图标状态 (close #862)   ← 干干净净就这一个提交
```

直接推送 → 开 PR → 维护者一眼看完，合并后无副作用。

#### 四个具体痛点

1. **Cherry-pick 容易冲突** — `pleggen_workspace` 上如果改了**与 PR 相关的同一文件**，cherry-pick 时一片红
2. **历史回溯困难** — 维护者 review PR 时看的是**这个分支的所有提交**，私人 commit 会出现在文件变更里
3. **容易把私人配置泄露出去** — 即使 `.gitignore` 忽略了，**已 commit 的文件**还是会推送
4. **心理负担重** — 在 `pleggen_workspace` 上会下意识担心"这个改动会不会污染 PR" → **降低开发效率**

#### 类比

- `pleggen_workspace` = **草稿本**（撕了也不心疼）
- `fix/xxx` = **正式答题卡**（写完直接交）

#### 例外情况

如果"私人改动"后来发现**对项目也有用**（比如 CLAUDE.md），可以单独开分支提 PR：

```bash
git checkout main
git checkout -b docs/add-claude-md upstream/dev
git cherry-pick pleggen_workspace   # 摘取该 commit
git push origin docs/add-claude-md
```

但**默认情况下，私人改动就别考虑提 PR 了**。

## PR 禁忌清单（绝不能进 PR）

- `.claude/` 目录
- `docs/` 目录（个人贡献笔记）
- `CLAUDE.md`
- `settings.local.json`
- 个人 `.gitignore` 改动（除非必要且在 PR 描述中说明）

## PR 后续工作

### 追加 commit 到已开的 PR

PR 是绑定到 head 分支的，不需要重新开 PR：

```bash
git checkout fix/<issue-num>-<short-desc>  # 确保在 fix 分支上
# 编辑代码
git add .
git commit -m "fix(backlog): address review feedback"
git push origin fix/<issue-num>-<short-desc>  # PR 自动更新
```

### Rebase 到最新 dev（PR 列表里清理过时 base）

如果 PR review 期间 `upstream/dev` 更新了，需要 rebase：

```bash
git fetch upstream
git rebase upstream/dev
# 处理可能的冲突
git push --force-with-lease=refs/heads/fix/<branch>:<old-sha> \
  origin fix/<issue-num>-<short-desc>
```

**`--force-with-lease=<refname>:<expected>` 比 `--force` 安全**：
- 显式指定"远端应该是这个 SHA"
- 如果远端被别的 push 改了，会拒绝
- 普通 `--force-with-lease` 在 rebase 后可能报"stale info"，**显式指定可绕过**

### ⚠️ Force push 的安全边界

- **对**自己的 fork **安全**（你是唯一能 push 的）
- **对**上游 `main` / `dev` **绝对不要**（除非有管理员权限）
- 自己的 PR 分支 force push 是**常见且推荐**的做法

## 私人改动同步到 main

如果 `pleggen_workspace` 上的某些实验性改动想保留到 `main`（但不提 PR）：

```bash
# 在 main 上摘取指定 commit
git checkout main
git cherry-pick <commit-hash>           # 单个
git cherry-pick <hash1>..<hashN>        # 多个连续
```

冲突时手动解决 → `git add .` → `git cherry-pick --continue`。

## 不提交到任何分支的工作区改动

有时想保留文件但暂不 commit：
- 工作区文件会保留在 `pleggen_workspace`（`git status` 显示 `??`）
- 切换分支时**不会丢失**（只要不 `git stash` 或 `git checkout -- <file>`）
- 但要小心：`git checkout` 到其他分支时，未追踪的同名文件可能被覆盖

## 推荐的 Git Alias（可选）

添加到 `~/.gitconfig`：

```ini
[alias]
    cps  = cherry-pick
    cb   = checkout -b
    co   = checkout
    br   = branch
    st   = status -sb
    ll   = log --oneline --graph -20
```

这样 `git cps <hash>` 比 `git cherry-pick <hash>` 顺手。

## 经验教训（来自 #866 PR）

| 经验 | 说明 |
|------|------|
| **PR base 必须是 `dev` 不是 `main`** | 见上方"⚠️ 重要"小节 |
| **首次配置 `upstream` remote** | 你的 fork 仓库只配了 `origin`，需要手动加 `upstream` |
| **commit message 用 Conventional Commits** | 项目里 review bot 会检查 |
| **测试后保持测试场景文件** | 方便回滚或后续 rebase 时验证 |
| **token 及时撤销** | push 完立刻去 GitHub Settings 撤销，避免长期泄露 |
| **force push 用 `--force-with-lease=<ref>:<expected>`** | rebase 后普通 `--force-with-lease` 会报 stale info |
