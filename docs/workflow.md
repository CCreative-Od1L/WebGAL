# 个人开发约定

## 分支策略

| 分支 | 用途 | 是否同步上游 |
|------|------|--------------|
| `pleggen_workspace` | 私人实验田：`.claude/`、`docs/`、`CLAUDE.md`、个人配置、实验性改动 | ❌ 不同步 |
| `main` (本地) | 干净的、和上游 `OpenWebGAL:main` 同步的主线 | ✅ 可提 PR |
| `OpenWebGAL/main` | 上游主分支 | `git pull` 拉更新 |

## 同步上游更新

```bash
git checkout main
git pull origin main
# 或
git fetch upstream
git merge upstream/main
```

## 提 PR 流程

1. **不要**在 `pleggen_workspace` 上直接做 Bug 修复
2. 切到 `main` → `git pull` 同步上游 → 拉新分支
   ```bash
   git checkout main
   git pull
   git checkout -b fix/<issue-num>-<short-desc>
   ```
3. 修改、提交（提交粒度要小：`fix: 简述 (close #xxx)`）
4. 推送到 fork
   ```bash
   git push origin fix/<issue-num>-<short-desc>
   ```
5. GitHub 网页上 Compare & Pull Request → Base: `OpenWebGAL/WebGAL:dev`

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
main → 拉出 fix/862-icon-reset
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
git checkout -b docs/add-claude-md
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
