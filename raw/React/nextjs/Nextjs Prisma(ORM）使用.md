# 使用 ORM 知识点（以 Prisma 为例）

> 本文以 **Next.js 16 为基线**、以 **Prisma 7（当前稳定版）为例**（最后更新 2026-10）。覆盖：为什么在 Next.js 里用 ORM、主流 ORM 对比、Prisma 安装与初始化、`schema.prisma` 模型与关系、迁移（migrate）、PrismaClient 单例模式、CRUD 查询（过滤/排序/分页/关联/事务/聚合）、在 Server Component / Route Handler / Server Action 中的用法、性能与生产最佳实践，附避坑速记清单与自测题。

## 版本说明

- **本笔记以 Prisma 7 为基线**（`@prisma/client` 7.x，经典 `schema.prisma` 写法）。这是当前**稳定、主流、生产可用**的版本。
- **Prisma 8 截至 2026-10 仍是 Release Candidate**（官方 2026-08-28 起 `prisma@latest` 指向 8，但 GA 预计 2026-10，API 可能变动）。Prisma 8 是"contract-first"全新架构：包改为 `@prisma/orm-postgres` / `@prisma/orm-mongo`，**不再读 `schema.prisma`，也没有 `generate` / `migrate dev` / `db push` 命令**，且暂缺 `$extends`、JSON 列过滤、原子 `increment`、多数嵌套写、事务隔离级别、`P2002` 风格错误码。**不采用 RC 版本作为生产基线。**
- Prisma 6 仅安全补丁维护至 2026-11-19。
- 想锁定 7：`package.json` 里 `"prisma": "^7"`（devDependencies）+ `"@prisma/client": "^7"`（dependencies）；CI/一次性命令用 `npx prisma@7`。
- 一句话理解：**ORM = 用类型安全的编程语言对象去操作数据库，把"表/行/SQL"抽象成"模型/对象/方法"，让应用代码不手写 SQL、编译期就能发现写错字段。**
- Next.js 16 相关提醒：**Turbopack 是默认打包器**，务必在 `next build` 之前先跑 `npx prisma generate`，否则服务端代码找不到生成的 client。

![Prisma 在 Next.js 中的架构与数据流](img/prisma-orm-architecture.svg)

---

## 一、为什么在 Next.js 里用 ORM

Next.js 是**全栈**框架：Server Component、Route Handler、Server Action 都是**运行在服务端**的代码，可以直接访问数据库。此时"数据访问"是一个高频、易错、重复的环节，ORM 的价值正在于此。

| 关注点 | 手写 SQL | ORM（Prisma） |
|---|---|---|
| 类型安全 | 查询结果多为 `any`，字段写错运行期才爆 | 模型即类型，写错字段编译期报错 |
| 代码量 | 大量模板 SQL + 手动映射 | `findMany({ where })` 一行搞定 |
| 字段/表改动 | 改 SQL + 改类型，容易漏 | 改 `schema.prisma` → 全局类型联动更新 |
| 迁移 | 自己维护 DDL | `prisma migrate` 生成并管理迁移 |
| 关联查询 | 手写 JOIN，易错 | `include` / `where: { relation: {...} }` |
| 注入风险 | 易拼 SQL 注入 | 参数化查询，默认安全 |

**与 Next.js 的关系**：ORM 只是"服务端数据访问层"，不参与渲染。它和 React Server Components 天然契合——服务端组件可以直接 `await prisma.post.findMany()`，把数据渲染成 HTML 下发，**不会把数据库连接或密钥打进浏览器 bundle**。

### 主流 ORM 横向对比（Node/TS 生态）

| ORM | 类型安全 | 活跃度/主流度 | 风格 | 说明 |
|---|---|---|---|---|
| **Prisma** | ★★★★★ | 极高（社区/文档最大） | Schema-first，自研 DSL | 本笔记主角；查询 API 优雅，生态最全 |
| **Drizzle ORM** | ★★★★★ | 高（增长快） | SQL-like，贴近 SQL | 轻量、性能好，无代码生成，学习曲线平缓 |
| **Kysely** | ★★★★ | 中高 | SQL Builder（非完整 ORM） | 适合想保留 SQL 手感又想要类型安全的场景 |
| TypeORM | ★★★☆ | 高（老牌） | 装饰器/Entity | 类与表映射，早期 NestJS 默认 |
| Sequelize | ★★★ | 高（老牌） | 模型实例 | 年代久远，类型体验一般 |
| MikroORM | ★★★★ | 中 | 实体/Unit of Work | 类 JPA 体验，灵活性高 |

> 选型建议（结合你一贯"用主流、社区活跃"的偏好）：**绝大多数 Next.js 项目选 Prisma 或 Drizzle**。二者都主流且类型安全；Prisma 胜在开箱即用与关系查询 API，Drizzle 胜在轻量与贴近 SQL。

---

## 二、安装与初始化

```bash
# 安装（Prisma 7，注意锁定版本）
npm install -D prisma@^7
npm install @prisma/client@^7

# 初始化：生成 prisma/schema.prisma 和 .env（含 DATABASE_URL 占位）
npx prisma@7 init
```

生成的目录结构：

```
my-app/
├─ prisma/
│  └─ schema.prisma     # 数据模型唯一事实来源
├─ .env                 # DATABASE_URL（已被 @next/env 加载）
└─ lib/prisma.ts        # PrismaClient 单例（手动创建，见第四节）
```

`.env` 示例：

```bash
# 以 PostgreSQL 为例
DATABASE_URL="postgresql://user:password@localhost:5432/mydb?schema=public"
```

> `schema.prisma` 里 `datasource` 的 `url = env("DATABASE_URL")`，说明连接串从环境变量读取——这也和 Next.js 的 `.env*` 机制（见本目录《nextjs 环境变量知识点.md》）衔接：**开发读 `.env`，生产从平台注入环境变量，代码里不写死连接串。**

---

## 三、`schema.prisma`：模型与关系

`schema.prisma` 是**数据模型的唯一事实来源**：改它 → `migrate` 同步表结构 + `generate` 重新生成类型 → 全项目类型联动。

```prisma title="prisma/schema.prisma"
// 1) 生成器：决定 client 长什么样
generator client {
  provider = "prisma-client-js"   // 经典/默认生成器，import 自 @prisma/client
}

// 2) 数据源：连接哪个库
datasource db {
  provider = "postgresql"          // postgresql | mysql | sqlite | mongodb ...
  url      = env("DATABASE_URL")
}

// 3) 模型：一张表 = 一个 model
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]                 // 反向关系：一个用户多篇帖子
  createdAt DateTime @default(now())
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
}
```

### 常用字段属性

| 属性 | 作用 |
|---|---|
| `@id` / `@id(map)` | 主键；复合主键用 `@@id([a, b])` |
| `@default(auto increment())` | 自增 |
| `@default(uuid())` / `cuid()` | 字符串主键常用 |
| `@unique` / `@@unique([a, b])` | 唯一约束 |
| `@relation(fields:[...], references:[...])` | 外键关联（字段需先定义） |
| `@db.VarChar(255)` 等 | 数据库原生类型微调 |
| `?` 后缀 | 可空字段 |
| `@updatedAt` | 更新时间自动维护 |

### 三种关系

| 关系 | 写法要点 |
|---|---|
| 1 : 1 | 一侧 `@unique` 的外键 + `@relation`；对侧为单对象字段 |
| 1 : N | 多侧持有外键字段 + `@relation(fields, references)`；一侧声明 `Child[]`（本示例 User↔Post） |
| N : N | 显式 join 表（`@@id([a,b])` + 双向数组）；或隐式（两侧各 `X[]` 相对，Prisma 自动建关联表） |

> 关系字段只存在于模型层，不在数据库里多占列；真正落库的是外键列（如 `authorId`）。

---

## 四、PrismaClient 单例（重点）

**为什么要单例**：Next.js 开发时热更新、生产时 serverless 都会反复创建模块。若每个文件都 `new PrismaClient()`，会开出一堆连接，把数据库连接池打爆（`P1001`/连接超时）。官方推荐：**全局只保留一个实例**。

```ts title="lib/prisma.ts"
import { PrismaClient } from "@prisma/client";

// 把实例挂到 globalThis，跨热更新/跨调用复用
const globalForPrisma = globalThis as unknown as { prisma?: PrismaClient };

export const prisma = globalForPrisma.prisma ?? new PrismaClient();

// 仅开发环境写入全局，避免热更新重复 new
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

**Driver Adapters（可选，生产/serverless 推荐）**：Prisma 7 支持用驱动适配器直连，如 PostgreSQL 用 `@prisma/adapter-pg`：

```ts title="lib/prisma.ts"
import { PrismaClient } from "@prisma/client";
import { PrismaPg } from "@prisma/adapter-pg";

const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL });
export const prisma = globalForPrisma.prisma ?? new PrismaClient({ adapter });
if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;
```

> 需要连接池（生产/serverless 下推荐）：数据库侧配 PgBouncer，或使用托管方案（如 Prisma Accelerate）。普通自建/单实例开发可先用默认直连。

---

## 五、迁移 Migrate（模型 → 数据库表）

```bash
# 开发：根据 schema.prisma 生成迁移并应用到数据库
npx prisma@7 migrate dev --name init

# 生产/部署：只应用已提交的迁移文件（无交互、幂等）
npx prisma@7 migrate deploy

# 生成客户端（改 schema 后重跑；build 前务必执行）
npx prisma@7 generate

# 不保留迁移历史，直接同步表结构（适合快速原型，不推荐生产）
npx prisma@7 db push

# 浏览器可视化查看/编辑数据
npx prisma@7 studio
```

`migrate dev` 会做三件事：①对比 schema 与数据库生成迁移 SQL；②执行迁移改库表；③调用 `generate` 更新 client 类型。

> **工作流闭环**：改 `schema.prisma` → `npx prisma migrate dev --name 描述`（同步表 + 生成 client）→ 代码里用新字段（类型已更新）。

---

## 六、CRUD 查询

`prisma` 上每个模型都是一个命名空间（`prisma.user`、`prisma.post`…），方法即 API。

### 创建 / 读取

```ts
// 创建
const user = await prisma.user.create({
  data: { email: "a@b.com", name: "Alice" },
});

// 读取全部（可按需裁剪字段，避免整行大对象）
const users = await prisma.user.findMany({
  select: { id: true, email: true },
});

// 按唯一字段取单条
const user = await prisma.user.findUnique({
  where: { email: "a@b.com" },
});
```

### 过滤、排序、分页

```ts
const posts = await prisma.post.findMany({
  where: {
    published: true,
    author: { name: { contains: "A" } }, // 关联条件过滤
    title: { contains: "Next.js" },
  },
  orderBy: { createdAt: "desc" },
  take: 10,
  skip: 0,
  // 大数据量推荐游标分页，翻页更快更稳：
  // cursor: { id: lastCursorId }, skip: 1, take: 10
});
```

### 关联查询（避免 N+1）

```ts
// include：一次查询带出关联数据，而不是循环里反复查库
const userWithPosts = await prisma.user.findUnique({
  where: { id: 1 },
  include: { posts: { where: { published: true } } },
});
```

### 更新 / 删除 / 幂等

```ts
await prisma.user.update({
  where: { id: 1 },
  data: { name: "Bob" },
});

// upsert：存在则更新，不存在则创建（幂等）
await prisma.user.upsert({
  where: { email: "a@b.com" },
  update: { name: "Bob" },
  create: { email: "a@b.com", name: "Bob" },
});

await prisma.post.deleteMany({ where: { published: false } });
```

### 事务与聚合

```ts
// 数组式：批量原子提交
await prisma.$transaction([
  prisma.user.update({ where: { id: 1 }, data: { name: "A" } }),
  prisma.post.create({ data: { title: "t", authorId: 1 } }),
]);

// 交互式：事务内做依赖当前值的逻辑
await prisma.$transaction(async (tx) => {
  const author = await tx.user.findUnique({ where: { id: 1 } });
  await tx.post.create({ data: { title: "t", authorId: author!.id } });
});

// 聚合：count / sum / avg / min / max
const stats = await prisma.post.aggregate({
  _count: { _all: true },
  _avg: { authorId: true },
});

// 需要手写 SQL 时用参数化查询（禁止字符串拼接用户输入）
const rows = await prisma.$queryRaw`SELECT * FROM "Post" WHERE title ILIKE ${keyword}`;
```

---

## 七、在 Next.js 中如何落地

### Server Component（服务端组件直接查）

```tsx title="app/posts/page.tsx"
import { prisma } from "@/lib/prisma";

export default async function PostsPage() {
  const posts = await prisma.post.findMany({
    where: { published: true },
    orderBy: { createdAt: "desc" },
    take: 20,
  });
  return (
    <ul>
      {posts.map((p) => <li key={p.id}>{p.title}</li>)}
    </ul>
  );
}
```

### Route Handler（API 路由）

```ts title="app/api/users/route.ts"
import { NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

export async function GET() {
  const users = await prisma.user.findMany({ select: { id: true, email: true } });
  return NextResponse.json(users);
}
```

### Server Action（表单/变更 + 缓存失效）

```ts title="app/actions.ts"
"use server";
import { prisma } from "@/lib/prisma";
import { revalidatePath } from "next/cache";

export async function createPost(formData: FormData) {
  await prisma.post.create({
    data: { title: formData.get("title") as string, authorId: 1 },
  });
  revalidatePath("/posts"); // 让服务端缓存失效，重新拉取最新数据
}
```

> **红线：不要在 Client Component（`"use client"`）里 import prisma**。客户端代码会被打进浏览器 bundle，等于把数据库访问逻辑和连接串暴露给用户。所有数据库操作走 Server Component / Route Handler / Server Action。

---

## 八、性能与生产最佳实践

1. **单例复用**（见第四节），否则 serverless 每次冷启动都建新连接 → 连接耗尽。
2. **只查需要的字段**：`select` / `take` 裁剪，避免全表 + 整行大对象拖垮内存。
3. **避免 N+1**：关联数据用 `include` / `where: { relation }` 一次取；不要在循环里逐条查询。
4. **大数据分页用游标**（`cursor`）而非 `skip`，深翻页更稳定。
5. **迁移在部署时执行**：`prisma migrate deploy`（只应用已提交迁移，幂等）；本地开发用 `migrate dev`。
6. **`next build` 前先 `generate`**（Next 16 默认 Turbopack）：在 `package.json` 加 `"postinstall": "prisma generate"` 或 CI 里先跑，避免构建报"找不到模型"。
7. **连接池**：生产/serverless 用驱动适配器 + PgBouncer，或托管加速方案。
8. **密钥安全**：`DATABASE_URL` 只放环境变量/平台注入，绝不提交到仓库、不硬编码进 bundle。

---

## 九、避坑速记清单

1. **忘记 `npx prisma generate`** → 类型对不上 / 报 "Cannot find module" / 新模型不存在。改 schema 后必跑 `generate`（`migrate dev` 已内含）。
2. **每个文件都 `new PrismaClient()`** → 连接池耗尽、热更新实例爆炸。用 `lib/prisma.ts` 全局单例。
3. **在 Client Component 里 import prisma** → 数据库逻辑/密钥进浏览器 bundle。全走服务端。
4. **改了 schema 只 `generate` 不 `migrate`** → 模型与表结构不一致，查询报错（表缺列）。
5. **`findMany` 不带 `where`/`select` 全量拉取** → 数据量大时内存与带宽爆炸。
6. **`$queryRaw` 字符串拼接用户输入** → SQL 注入。必须用模板参数 `\`...${x}\``。
7. **把 Prisma 8（RC）当生产基线** → API 未定稿、缺事务隔离/原子自增/`$extends` 等能力。生产锁定 Prisma 7。
8. **`npx prisma` 默认指到 8** → 7 项目会因"不再读 schema.prisma / 没有 migrate 命令"而报错。用 `npx prisma@7` 或锁定 `prisma@^7`。
9. **生产手跑 `migrate dev`** → 有交互、可能改坏库。部署一律 `migrate deploy`。
10. **`.env` 未配置 / 未加载** → 连接报错。确认 `DATABASE_URL` 存在且被 Next.js 读取（见环境变量笔记）。

---

## 十、自测题

**Q1.** 在 Next.js 里，为什么"不允许在 Client Component 里 import prisma"？正确的数据访问入口有哪些？

**Q2.** 开发环境热更新时报"数据库连接数过多 / 超时"，最可能的原因是什么？怎么修？

**Q3.** 我改了 `schema.prisma` 加了字段，但代码里还是报"该字段不存在"，该依次执行哪些命令？

**Q4.** 列表页想一次带出每篇文章的作者名，避免逐条查库，应该怎么写？这解决的是什么问题？

**Q5.** 部署到生产环境时，应该用哪个迁移命令？它和 `migrate dev` 有什么区别？

<details>
<summary>参考答案（点击展开）</summary>

**A1.** Client Component 代码会被打进浏览器 bundle，import prisma 等于把数据库访问逻辑和连接串暴露给用户。正确入口：Server Component 直接查询、Route Handler（`route.ts`）、Server Action（`"use server"`）。

**A2.** 最可能是每处 `new PrismaClient()` 在热更新时重复创建实例，连接数被耗尽。修法：用 `lib/prisma.ts` 全局单例（`globalForPrisma`），开发环境把实例挂到 `globalThis`，生产复用同一实例。

**A3.** 依次执行：`npx prisma migrate dev --name <描述>`（把新字段同步到数据库表 + 自动 generate），必要时单独 `npx prisma generate`。执行后 client 类型才包含新字段。

**A4.** 用 `include`：`prisma.post.findMany({ include: { author: true } })`。一次查询带出关联数据，解决 **N+1 问题**（避免在循环里对每篇文章各查一次作者）。

**A5.** 用 `npx prisma migrate deploy`。它只应用已提交的迁移文件、无交互、幂等；`migrate dev` 用于本地开发（会生成新迁移、有交互），不适合生产环境。
</details>

---

## 参考

- Prisma ORM 官方文档（version 7，`/orm/v7`）— https://www.prisma.io/docs/orm/v7
- Prisma 与 Next.js 集成指南 — https://www.prisma.io/docs/guides/frameworks/nextjs
- Prisma ORM Release Status（8 为 RC、7 为稳定）— https://www.prisma.io/docs/orm/release-status
- Next.js 16 发布说明 — https://nextjs.org/blog/next-16
- Prisma 7 Changelog / 8 说明 — https://www.prisma.io/changelog
