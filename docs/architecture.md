# 核心架构

## 双层渲染

```
┌─────────────────────────────────────┐
│ React DOM 层（UI 叠加）              │  ← Stage.tsx 中各子组件
│  - 文本框、菜单、回放、按钮          │
├─────────────────────────────────────┤
│ Pixi.js 层（WebGL 游戏内容）         │  ← PixiController 管理
│  - 背景、立绘、特效、Live2D、Spine   │
└─────────────────────────────────────┘
```

两者叠加在同一个 `<div>` 容器内，通过 React 协调。

## 关键目录

```
packages/webgal/src/
├── App.tsx              # 顶层组件，挂载所有 UI
├── main.tsx             # 入口：ReactDOM + i18n + Redux Provider
├── Core/                # 引擎核心（最常修改）
│   ├── WebGAL.ts        # 全局单例（WebGAL, Live2D）
│   ├── webgalCore.ts    # 核心类：场景/回退/动画/游戏流程管理
│   ├── initializeScript.ts # 引擎初始化
│   ├── Modules/         # 核心模块
│   │   ├── scene.ts     # 场景管理
│   │   ├── backlog.ts   # 回退日志
│   │   ├── animations.ts# 动画管理
│   │   ├── gamePlay.ts  # 运行时（自动/快进/跳过）
│   │   ├── events.ts    # 事件总线（基于 mitt）
│   │   ├── perform/     # 演出控制器
│   │   └── stage/       # 舞台状态管理
│   ├── controller/      # 控制器
│   │   ├── gamePlay/    # autoPlay / fastSkip
│   │   ├── scene/       # 场景加载
│   │   ├── stage/       # Pixi 舞台同步
│   │   └── storage/     # 存档读档
│   ├── parser/          # 桥接 packages/parser 的场景解析
│   ├── gameScripts/     # 50+ 脚本指令实现 ⭐
│   ├── integration/     # 第三方集成（Steam 等）
│   └── util/            # 工具函数
├── Stage/               # Pixi 舞台 UI 容器
│   ├── Stage.tsx
│   ├── FigureContainer/ # 立绘
│   ├── AudioContainer/  # 音频
│   ├── TextBox/         # 文本框
│   └── FullScreenPerform/
├── UI/                  # 纯 React UI
│   ├── Menu/ Backlog/ Title/ Extra/ DevPanel/ ...
├── store/               # Redux store（GUI / userData / saveData）
├── translations/        # 8 种语言
└── config/              # 全局配置
```

## 脚本执行流

```
.txt 场景文件
   ↓ packages/parser (Chevrotain 解析)
SceneStatement[]  (语句数组)
   ↓ packages/webgal/src/Core/parser/sceneParser.ts
按序调用 gameScripts/ 下对应指令
   ↓ 修改 stageState + 派发 perform
Pixi 舞台 + React UI 同步刷新
```

## 重点文件（修改时优先看这里）

| 指令类型 | 路径 |
|----------|------|
| 对话 | `Core/gameScripts/say.ts` |
| 背景切换 | `Core/gameScripts/changeBg/` |
| 立绘切换 | `Core/gameScripts/changeFigure.ts` |
| 变换 | `Core/gameScripts/setTransform.ts` |
| 动画 | `Core/gameScripts/setAnimation.ts` |
| 选项 | `Core/gameScripts/choose/` |

## 状态管理分层

- **运行时游戏状态**（场景、立绘、变量等）→ `WebGAL.sceneManager.*` / `stageStateManager`
- **UI 状态**（菜单展开、按钮按下等）→ Redux store
- **跨模块通信** → `mitt` 事件总线（避免循环依赖）
- **持久化**（存档、用户偏好）→ localforage
