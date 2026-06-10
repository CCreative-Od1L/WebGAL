# 项目速览

## WebGAL 是什么

**WebGAL** 是一款**网页端视觉小说（Visual Novel / Galgame）引擎**，使用 MPL-2.0 协议开源。

- 当前版本：v4.6.1
- 目标用户：创作者（无需编程基础）、开发者（可深度定制）
- 配套工具：[WebGAL Terre](https://github.com/OpenWebGAL/WebGAL_Terre)（官方图形化编辑器）
- 在线演示：<https://demo.openwebgal.com>

## 核心特点

- **一次编写、跨平台运行**：Web 端打包可生成 Win/macOS/Linux 可执行文件
- **自研脚本语言**：.txt 格式，类自然语言
- **双层渲染架构**：React + Pixi.js 叠加
- **完整 VN 特性**：立绘、背景、Live2D、Spine、视频、CG 鉴赏、存档/读档

## 技术栈

| 层级 | 选型 |
|------|------|
| 前端框架 | React 17 + TypeScript |
| 渲染引擎 | Pixi.js 6（WebGL 2D 渲染 + Live2D） |
| 状态管理 | Redux Toolkit |
| 样式 | SCSS + Emotion CSS-in-JS |
| 国际化 | i18next（8 种语言） |
| 动画补间 | popmotion |
| 持久化 | localforage（IndexedDB 封装） |
| 构建工具 | Vite 5（webgal）+ Rollup（parser） |
| 包管理 | Yarn 1.22 Workspaces（Monorepo） |
| 脚本解析 | Chevrotain（自研 DSL） |
| 本地服务器 | Express（packages/server） |

## Monorepo 结构

```
WebGAL/
├── packages/
│   ├── webgal/         # 核心引擎（React + Pixi.js 前端）— 主要工作目录
│   ├── parser/         # 独立 npm 包：WebGAL 脚本解析器
│   ├── server/         # 本地 HTTP 服务器（含预编译二进制）
│   └── yukimi/         # Yukimi 脚本编译器工具链
├── dev-docs/           # 项目自带的中文设计文档（可上游同步）
├── docs/               # 个人贡献笔记（私人维护，不同步上游）
└── package.json        # Workspace 根配置
```

## 注意：不要混淆两个 docs 目录

- **`dev-docs/`** — 项目自带的**设计文档**（动画系统、UI 层级等），可上游同步
- **`docs/`** — **私人贡献笔记**（CLAUDE.md 拆分内容、Issue 整理等），只留在 fork
