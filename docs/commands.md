# 常用命令

## 根目录命令

| 命令 | 作用 |
|------|------|
| `yarn dev` | 启动开发服务器（端口 3000）— 先 build parser，再启动 webgal |
| `yarn build` | 完整构建（parser + webgal） |
| `yarn build-ci` | CI 模式构建 |
| `yarn parser:test` | 运行 parser 单元测试 |
| `yarn parser:test-coverage` | parser 测试覆盖率 |
| `yarn parser:build` | 只构建 parser |
| `yarn webgal:dev` | 只启动 webgal 包开发模式 |
| `yarn webgal:build` | 只构建 webgal |

## 开发流程

```bash
# 1. 首次启动（会先构建 parser）
yarn dev

# 2. 后续改动 webgal 代码会自动热更新
# 3. 如果改了 packages/parser，需要重新 build
yarn parser:build
```

## packages/webgal 单独命令

进入 `packages/webgal/` 后可单独运行：

| 命令 | 作用 |
|------|------|
| `yarn dev` | Vite dev server（端口 3000，`--host` 暴露局域网） |
| `yarn build` | 生产构建（base=`./`，适合静态部署） |
| `yarn preview` | 预览构建产物 |
| `yarn lint` | ESLint 检查并自动修复 |

## packages/parser 单独命令

| 命令 | 作用 |
|------|------|
| `yarn test` | vitest 监听模式 |
| `yarn coverage` | vitest 单次运行 + c8 覆盖率 |
| `yarn build` | Rollup 构建（CJS + ES + Types） |
| `yarn debug` | 调试入口 `test/debug.ts` |
| `yarn debug-scss-parser` | 调试 SCSS 解析器 |
| `yarn debug-linebreak-parser` | 调试换行解析器 |

## 环境要求

- Node ≥ 18
- Yarn 1.22.22（`packageManager` 字段锁定）
