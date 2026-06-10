# 与 AI 助手协作的建议

## 阅读优先级

当被问及 WebGAL 的修改时，AI 助手应：

1. **优先读 `Core/gameScripts/`** — 50+ 脚本指令的实现都在这里
2. **其次看 `Core/Modules/`** — 理解场景/回退/动画/事件等核心机制
3. **舞台相关**：同时看 `Stage/` + `Core/controller/stage/`
4. **UI 相关**：看 `UI/` 目录
5. **脚本解析相关**：看 `packages/parser/`

## 修改禁区

- **`packages/parser/` 慎动** — 那是独立 npm 包，改动需要同步发包，影响面大
- **不要重构架构** — 在不熟悉前不要重写事件总线或状态管理
- **不要修改发布配置** — `vite.config.ts`、`tsconfig.json`、`packages/parser/rollup.config.js`

## 修改前先做的检查

- [ ] 看了 `docs/architecture.md` 理解整体结构
- [ ] 找到对应功能所在的文件
- [ ] 检查是否有相关测试（`packages/parser/test/`）
- [ ] 确认改动不会影响 .gitignore、个人配置

## 改 Bug 后的好习惯

- 在 commit message 中 `close #<num>` 自动关闭 Issue
- 如能帮助后续维护者，在 `dev-docs/` 写一条简短说明
- 不确定时**先问**，不要假设意图（特别是跨模块改动时）

## 高效提问模板

问 AI 助手时尽量包含：

```
- 我要改的功能/修的 Bug：<一句话>
- Issue 链接（如有）：#<num>
- 期望行为：<具体描述>
- 当前行为：<实际表现>
- 我已经看过的文件：<路径>
```

## 不要问的

- "帮我看看这个项目能干什么" → 直接看 [project-overview.md](./project-overview.md)
- "怎么用 cherry-pick" → 直接看 [workflow.md](./workflow.md)
- "哪些 Issue 适合新手" → 直接看 [issues-screening.md](./issues-screening.md)
