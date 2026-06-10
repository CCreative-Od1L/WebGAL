# 命名与代码约定

## 文件命名

- **React 组件**：PascalCase → `Stage.tsx`、`TextBox.tsx`
- **脚本指令 / 工具**：camelCase → `say.ts`、`changeFigure.ts`
- **多文件目录**（含子文件）：camelCase 目录 + PascalCase 子文件 → `changeBg/index.ts` + `next.ts`

## 路径别名

- `@/` → `src/`（在 `vite.config.ts` 中配置）
- 例：`import { ... } from '@/Core/WebGAL'`

## 状态管理约定

| 用途 | 使用方式 |
|------|----------|
| 运行时游戏状态 | `WebGAL.sceneManager.*` / `stageStateManager` |
| UI 状态 | Redux store（GUI / userData / saveData） |
| 跨模块通信 | `mitt` 事件总线 |
| 持久化 | localforage |

## 样式约定

- **UI 组件**：使用 `*.module.scss`（CSS Modules）
- **演出特效**：直接用 SCSS 或 Emotion `css`
- **样式顺序问题**：注意 `updateStyle` 刷新模板时 key 顺序不稳定（见 [issues-screening.md §#841](./issues-screening.md)）

## TypeScript

- 严格度较高，新代码必须带类型
- 避免 `any`，必要时用 `unknown` 收窄
- 接口定义集中在 `interface/` 下

## 提交信息约定（Conventional Commits v1.0）

采用 [Conventional Commits 1.0](https://www.conventionalcommits.org/zh-hans/v1.0.0/) 规范。

### 格式

```
<type>[optional scope][!]: <description>

[optional body]

[optional footer(s)]
```

### Type（必填）

| Type | 说明 | SemVer 映射 |
|------|------|-------------|
| `feat` | 新功能 | MINOR |
| `fix` | Bug 修复 | PATCH |
| `refactor` | 重构（非新功能、非 Bug） | — |
| `perf` | 性能优化 | PATCH |
| `docs` | 文档变更 | — |
| `style` | 代码格式（不影响逻辑） | — |
| `test` | 测试新增或修改 | — |
| `build` | 构建系统、依赖变更 | — |
| `ci` | CI 配置变更 | — |
| `chore` | 其他不修改 src 或 test 的变更 | — |
| `revert` | 回滚某次提交 | — |

### Scope（可选但推荐）

表示影响的范围，用括号包裹。

**本项目常用 scope**：
- `webgal` / `parser` / `server` — 包名
- `core` / `stage` / `ui` / `store` — 引擎子模块
- `i18n` / `build` / `lint` — 通用范畴
- 组件/文件名（如 `backlog`、`textbox`、`pixi`）— 精确范围

### Description（必填）

- 简明描述变更
- 50 字符以内最佳，不超过 72
- 使用中文或英文均可，**保持一致**
- 首字母小写，结尾不加句号
- 祈使语气（"add" 而非 "added"）

### Body（可选）

- 与 subject 间空一行
- 详细说明"**为什么**"改，而不是"**做了什么**"
- 可分多行

### Footer（可选）

- 与 body 间空一行
- 常用关键字：
  - `Refs #123` / `Closes #123` / `Fixes #123` — 关联 Issue
  - `BREAKING CHANGE: <描述>` — 不兼容变更（也可在 type/scope 后加 `!` 标记）
  - `Reviewed-by: <name>`
  - `Co-authored-by: <name>`

### BREAKING CHANGE

不兼容变更必须：
1. 在 type/scope 后加 `!`：`feat(api)!: remove legacy parser`
2. **或** 在 footer 写 `BREAKING CHANGE: <描述>`

### 完整示例

```
feat(webgal): add ignoreDefault argument to animation commands

允许 changeBg / changeFigure / setTransition 等指令通过
ignoreDefault 参数跳过未声明的默认变换和效果。
解决立绘 enter 动画覆盖 transform 的问题。

Refs #846
```

```
fix(pixi): reset fast-forward button pressed state on game end

end; 后未重置按钮的 pressed class，导致重启游戏时按钮仍
处于"按下"视觉状态。功能正常，仅样式残留。

Closes #862
```

```
refactor(core)!: rewrite event bus with typed events

BREAKING CHANGE: mitt 事件类型不再使用字符串字面量，必须
改用 SceneEvents 中预定义的常量。迁移成本见 dev-docs/。
```

### 提交粒度

- 一个 commit 只做一件事
- 同一文件的多处不相关修改 → 拆成多个 commit（用 `git add -p`）
- 修复 + 重构混在一起 → 拆（先重构单独 commit，再修 Bug）

### PR 标题

PR 标题 = **squash merge 时将采用的 commit 信息**，必须遵循同一规范。

---

## 未来增强：机械化强制规范

> 此章节记录**待评估的强化方案**，当前**未实施**。当维护压力出现时再启用。

### 候选方案

#### 方案 A：commitlint + husky（强制式）

提交时自动检查，不符合规范直接拒绝。

**组件**：
- `@commitlint/cli` + `@commitlint/config-conventional` — 规范检查
- `husky` — Git 钩子管理器

**安装**：
```bash
yarn add -D @commitlint/cli @commitlint/config-conventional husky
cat > commitlint.config.js << 'EOF'
module.exports = { extends: ['@commitlint/config-conventional'] };
EOF
yarn husky init
yarn husky add .husky/commit-msg 'yarn commitlint --edit "$1"'
```

**优点**：强制性强，无法绕过
**缺点**：增加依赖；**不能提交到上游**；老 commit 不受影响但已不规范

**绕过**：`git commit --no-verify -m "..."`

#### 方案 B：commitizen（向导式）

不强制，但提供交互式提交向导（选 type → 填 scope → 写 subject）。

**安装**：
```bash
npx commitizen init cz-conventional-changelog --save-dev --save-exact
```

**使用**：`git cz` 代替 `git commit`

**优点**：轻量，不强制，适合探索期
**缺点**：没有强制力

### 启用时机

满足以下任一条件时再考虑启用方案 A：
- 开始规律性地给上游提 PR
- 邀请他人加入 fork
- 多次发现自己不自觉地写不规范 commit

当前阶段（个人探索 fork）**不推荐**启用，理由：
1. 节奏优先于规范
2. 已能自觉执行（9 个 commit 全合规）
3. 加 husky 后未来 cherry-pick 老 commit 时可能踩坑

### 注意事项

- 这些工具的**配置文件和依赖不提交到上游**
- 如果未来真的需要，单独在一个 commit 里加，并标记为 `chore`
- 考虑使用 `.husky/` 的 `.gitignore` 处理（虽然本项目目前不用）

## 注释约定

- 文件顶部用 JSDoc `@file` 说明文件用途
- 复杂函数用 `@param` `@returns`
- 中文注释 OK，但避免大段解释代码做什么（应让代码自解释）
