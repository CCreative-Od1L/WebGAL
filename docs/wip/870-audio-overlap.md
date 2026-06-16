# WIP: #870 playEffect 快进音效重叠

> **状态**：⏸ 暂停 — 等待 GitHub token 完成 push 和开 PR
> **创建**：2026-06-16
> **关联 Issue**：[OpenWebGAL/WebGAL#870](https://github.com/OpenWebGAL/WebGAL/issues/870)

---

## 当前快照

| 项目 | 状态 |
|------|------|
| **分支** | `fix/870-audio-overlap-fast`（基于 `upstream/dev`）|
| **本地 commit** | `8a84c5a0` — `fix(webgal): prevent playEffect from creating duplicate audio on re-entry` |
| **修改文件** | `packages/webgal/src/Core/gameScripts/playEffect.ts` (+8 lines) |
| **TypeScript 验证** | ✅ 通过（`tsc --noEmit` 无错误）|
| **Push 到 fork** | ⏳ 缺 GitHub PAT |
| **PR** | ⏳ 等待 push 后开 |

---

## 任务清单

- [x] 阅读 #870 Issue 详情
- [x] 探索相关源码（playEffect / performController / fastSkip / useHotkey / sceneParser）
- [x] 定位 root cause
- [x] 建立工作分支（`fix/870-audio-overlap-fast` from `upstream/dev`）
- [x] 实现修复（playEffect.ts startFunction 加 guard）
- [x] 本地验证（`tsc --noEmit` 通过）
- [x] 提交
- [ ] **推送**（缺 token）
- [ ] **开 PR**

---

## 根因分析

### 直接原因

`playEffect` 的 `startFunction` 每次调用都 `document.createElement('audio') + play()`。如果同一 perform 的 `startFunction` 被重复调用（快进 + 动画的边缘时序、编辑器 live preview 等），旧的 `<audio>` 元素引用被新元素覆盖、成为孤儿继续播放，叠加新元素即可听到"重叠"。

### 历史背景

旧版本（pre-RFC1，4.5.x）的 `playEffect` 存在更严重的真 bug：

```typescript
// 旧版（pre-RFC1）playEffect 末尾
return {
  performName: 'none',  // ← 包装 performName 是 'none'
  ...
  arrangePerformPromise: new Promise((resolve) => {
    setTimeout(() => {
      const perform: IPerform = {
        performName: performInitName,  // ← 实际是 'effect-sound'
        ...
      };
      resolve(perform);
      seElement?.play();
    }, 1);
  }),
};
```

旧代码开头调 `unmountPerform('effect-sound', true)` 找旧 perform，但**彼时旧 perform 的 name 还是 `'none'`**（resolve 还没发生）→ dedup 找不到 → 旧音频永远不会被 stop → 与新音频叠加。

**RFC1 重构（2026-04-28，commit `cba2e22a`）后**：`playEffect` 直接返回 `performName: 'effect-sound'`，dedup 路径正确。pre-RFC1 的真 bug 在新代码下已修复。

### 当前修复

防御性 guard（RFC1 后的安全网）：

```typescript
startFunction: () => {
  if (seElement) {
    return;  // 已存在音频元素，跳过创建
  }
  let volume = getNumberArgByKey(sentence, 'volume') ?? 100;
  volume = Math.max(0, Math.min(volume, 100));
  seElement = document.createElement('audio');
  ...
}
```

`seElement` 在 `stopFunction` 中会被置 `null`，所以**不影响正常重播流程**；只在"未 stop 就重新 start"这种异常情况下兜底。

---

## 继续流程（下次会话直接照做）

### 1. 切换到 fix 分支

```bash
cd /home/pleGGen/projects/WebGAL
git checkout fix/870-audio-overlap-fast
```

### 2. 推送

按 workflow.md 的"token 及时撤销"原则：

```bash
# 生成 PAT: GitHub → Settings → Developer settings → Personal access tokens (Fine-grained)
#   Permissions: Contents (Read and Write), Pull requests (Read and Write)
# 仓库范围: 限定到 CCreative-Od1L/WebGAL

# 推送（用 token URL）
git push https://<YOUR_TOKEN>@github.com/CCreative-Od1L/WebGAL.git fix/870-audio-overlap-fast

# push 完立即去 GitHub 撤销该 token
```

### 3. 开 PR

URL：
```
https://github.com/OpenWebGAL/WebGAL/compare/dev...CCreative-Od1L:fix/870-audio-overlap-fast?expand=1
```

| 字段 | 值 |
|------|-----|
| **Base** | `OpenWebGAL/WebGAL:dev` |
| **Head** | `CCreative-Od1L:fix/870-audio-overlap-fast` |
| **Title** | `fix(webgal): prevent playEffect from creating duplicate audio on re-entry` |

PR 描述（已就绪，复制粘贴即可）：

```markdown
## 修复内容

修复 #870 描述的 `playEffect` 音效在快进 + 动画场景下重叠播放的问题。

## 根因

`playEffect` 的 `startFunction` 在每次调用时都 `document.createElement('audio')` + `play()`。如果同一 perform 的 `startFunction` 被重复调用（如快进 + 动画的边缘时序、编辑器 live preview 等），旧的 `<audio>` 元素引用被新元素覆盖、成为孤儿继续播放，叠加新元素即可听到"重叠"。

## 修复

在 `startFunction` 开头加 guard：`if (seElement) return;`。`seElement` 在 `stopFunction` 中会被置 `null`，所以不影响正常重播流程；只在"未 stop 就重新 start"这种异常情况下兜底。

## 备注

旧版本（pre-RFC1，4.5.x）的 playEffect 存在更严重的真 bug（`performName: 'none'` 包装 + `setTimeout` 异步 resolve），RFC1 重构后该路径已修复。本 PR 是防御性的安全网，对正常播放流程无影响。

Refs #870
```

---

## 关键 commit

```
8a84c5a0 fix(webgal): prevent playEffect from creating duplicate audio on re-entry
```

完整 commit message：

```
fix(webgal): prevent playEffect from creating duplicate audio on re-entry

在 playEffect 的 startFunction 开头加防御性 guard：若 seElement 已存在则直接 return。

避免在某些边缘场景下（快进 + 动画、编辑器 live preview 等）
startFunction 被重复调用时创建多个 <audio> 元素，导致旧音频
成为孤儿继续播放、与新音频叠加出现重叠症状。

旧版本（pre-RFC1）存在更严重的真 bug：performName 在 setTimeout
中才被 resolve 为 'effect-sound'，开头 unmountPerform 找不到旧
perform（彼时 name 还是 'none'），导致旧音频永远无法被 stop。
RFC1 重构后该路径已被修复，但作为安全网保留此 guard。

Refs #870
```

---

## 探索时参考的源码（后续可能还需要）

- `packages/webgal/src/Core/gameScripts/playEffect.ts` — 修复目标
- `packages/webgal/src/Core/Modules/perform/performController.ts` — 演出管理，dedup 逻辑
- `packages/webgal/src/Core/controller/gamePlay/fastSkip.ts` — 快进控制
- `packages/webgal/src/Core/controller/gamePlay/nextSentence.ts` — 步进函数
- `packages/webgal/src/Core/controller/gamePlay/autoPlay.ts` — 自动播放
- `packages/webgal/src/Core/controller/gamePlay/scriptExecutor.ts` — 脚本执行
- `packages/webgal/src/hooks/useHotkey.tsx` — 键盘/鼠标热键（Ctrl、滚轮）
- `packages/parser/src/scriptParser/commandParser.ts` — `addNextArg`（playEffect 隐式 -next）
- `packages/webgal/src/Core/parser/sceneParser.ts` — 脚本命令注册

---

## 相关链接

- Issue: https://github.com/OpenWebGAL/WebGAL/issues/870
- RFC1 refactor: commit `cba2e22a` (2026-04-28)
- Fast-forward 稳定化修复: commit `b617cf22` (2026-05-29)
- 旧 audio end handling 修复: commit `846c4230`
