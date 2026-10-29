# 2. Workspaces 包管理

> 是以 `package.json` 的 workspaces 管理包的一种工具，会将 workspaces 里的目录在初始化的时候以 `npm link` 的形式注册到 `.bin` 中，在使用时，也是使用 workspaces 目录里的包，而不是 node_modules 里面的包。

这段理解是对的，展开讲。

## 三种 workspaces 实现对比

| 工具 | 配置文件 | 磁盘占用 | 依赖隔离 | 推荐度 |
|---|---|---|---|---|
| npm workspaces | 根 `package.json` 的 `"workspaces"` 字段 | 一般（hoisting） | 弱（幻影依赖易出现） | 小型 |
| Yarn workspaces | 根 `package.json` 的 `"workspaces"` 字段 | 一般 | 弱 | 老项目 |
| **pnpm workspaces** | `pnpm-workspace.yaml` | **硬链接，省磁盘** | **强（默认严格，杜绝幻影依赖）** | **主流推荐** |

### pnpm-workspace.yaml

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

### npm / Yarn workspaces（根 package.json）

```jsonc
{
  "name": "acme-monorepo",
  "private": true,
  "workspaces": ["apps/*", "packages/*"]
}
```

## symlink 软链机制

当你在根目录执行 `pnpm install`，包管理器做的事：

1. 扫描 `pnpm-workspace.yaml` 里声明的 `packages/*`
2. 把每个内部包注册到根 `node_modules` 里，**以 symlink（软链接）形式指向 `packages/` 源码目录**
3. 包的 `bin` 命令同时链接到 `node_modules/.bin/`

```
node_modules/
├── @acme/
│   ├── ui    -> ../../packages/ui      （软链，不是真拷贝）
│   └── utils -> ../../packages/utils   （软链）
└── .bin/
    ├── ui --watch -> ../../packages/ui/dist/cli.js
    └── ...
```

**好处**：你改 `packages/utils` 的源码，`apps/web` 里 `import { xxx } from '@acme/utils'` 立刻生效——不需要 `npm publish`、不需要 `npm link` 手动操作。这就是 monorepo 内部开发体验的核心。

## `workspace:*` 协议

内部包互相引用时，不要写死版本号：

```jsonc
// packages/ui/package.json
{
  "dependencies": {
    "@acme/utils": "workspace:*"   // ✅ 永远指向本地源码
  }
}
```

发布时由 Changesets / pnpm 自动把 `workspace:*` 替换成实际版本号（如 `^1.2.0`）。

pnpm 还支持 `workspace:^`、`workspace:~`，分别对应发布时替换为 `^x.y.z`、`~x.y.z`。

## hoisting（依赖提升）

默认情况下，包管理器会把公共依赖提升到根 `node_modules`，避免重复安装：

```
packages/ui/node_modules/react     ❌ 不会真的在这里
node_modules/react                 ✅ 提升到根，所有包共享
```

**hoisting 的副作用**：幻影依赖。代码里 `import _ from 'lodash'` 但本包 `package.json` 没声明 `lodash`，靠 hoisting 侥幸跑通——pnpm 默认严格模式会直接报错。

pnpm 也提供 `.npmrc` 配置调整 hoisting：

```ini
# .npmrc
shamefully-hoist=false        # 默认 false，严格模式
# shamefully-hoist=true      # 老项目迁移时临时放开
strict-peer-dependencies=true
auto-install-peers=true
```

## pnpm catalog（统一版本号）

多个包都要用 `react@18`、`typescript@5.4`，不要每个包写一遍：

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'

catalog:
  react: ^18.3.0
  react-dom: ^18.3.0
  typescript: ^5.4.0
  eslint: ^9.0.0
```

```jsonc
// 任意一个包
{
  "dependencies": {
    "react": "catalog:"
  },
  "devDependencies": {
    "typescript": "catalog:",
    "eslint": "catalog:"
  }
}
```

升级版本时只改 `pnpm-workspace.yaml` 一处。

## `overrides` / `resolutions`

强制统一某个依赖的版本（比如所有包都用同一个 `lodash`，避免重复打包）：

```jsonc
// 根 package.json（npm）或 pnpm-workspace.yaml（pnpm）
{
  "pnpm": {
    "overrides": {
      "lodash": "4.17.21",
      "react": "$react"      // 引用根依赖里的 react 版本
    }
  }
}
```

Yarn 对应字段叫 `resolutions`。

## 常用命令

```bash
# 安装整个仓库依赖
pnpm install

# 给某个包加依赖
pnpm --filter @acme/ui add lodash

# 在某个包里跑 script
pnpm --filter @acme/ui dev

# 在所有包里跑 script
pnpm -r run build

# 看依赖图
pnpm list --depth 1
pnpm why lodash
```

`--filter` 是 monorepo 日常最高频的命令：
- `--filter=@acme/ui` 只操作这个包
- `--filter=@acme/ui...` 这个包 + 它的所有依赖
- `--filter=@acme/ui...^` 这个包 + 依赖它的所有下游
