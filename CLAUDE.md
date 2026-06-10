# CLAUDE.md

> AI 协作指南（精简版）。详细内容按主题拆分到 [docs/](./docs/)，本文件仅保留速查索引。
>
> 维护者：@pleGGen · 创建：2026-06-10

---

## 项目一句话速览

**WebGAL** = 网页端 Galgame 引擎 · React 17 + Pixi.js 6 · v4.6.1 · MPL-2.0 · Yarn Workspaces Monorepo。

---

## 文档导航

| 文档 | 内容 |
|------|------|
| [docs/project-overview.md](./docs/project-overview.md) | 项目速览、技术栈、Monorepo 结构 |
| [docs/architecture.md](./docs/architecture.md) | 核心架构：双层渲染、目录树、脚本执行流 |
| [docs/commands.md](./docs/commands.md) | 常用命令 |
| [docs/conventions.md](./docs/conventions.md) | 命名与代码约定 |
| [docs/workflow.md](./docs/workflow.md) | **个人开发约定**（分支策略、提 PR 流程、cherry-pick 同步） |
| [docs/issues-screening.md](./docs/issues-screening.md) | 开放 Issue 筛选清单（按难度分级） |
| [docs/ai-collaboration.md](./docs/ai-collaboration.md) | 与 AI 助手协作的建议 |

> 速查用本文件；深入了解时跳到具体文档。

---

## 三条铁律

1. **私人配置留在 `pleggen_workspace`**：`.claude/`、`docs/`、`CLAUDE.md`、`settings.local.json` 绝不进 PR
2. **修 Bug → 从干净的 `main` 拉分支**，不直接在 `pleggen_workspace` 上做
3. **`packages/parser/` 慎动**：它是独立 npm 包，改动需同步发包

---

## 关键路径速记

```
引擎入口    packages/webgal/src/main.tsx
核心类      packages/webgal/src/Core/WebGAL.ts
脚本指令    packages/webgal/src/Core/gameScripts/<command>.ts
Pixi 舞台   packages/webgal/src/Core/controller/stage/pixi/PixiController.ts
React UI    packages/webgal/src/UI/
Redux       packages/webgal/src/store/
脚本解析    packages/parser/src/
```

---

*最后更新：2026-06-10*
