# Next.js 路由知识点

> App Router 为主（Next.js 13+ 默认），并对比旧版 Pages Router。涵盖文件系统路由、动态/并行/拦截路由、导航、Hook API、中间件与 Route Handlers，附避坑速记清单与自测题。

## 版本说明

- **本文以 Next.js 16 为基线**（App Router）。所有代码示例均按 16 的写法。
- 与 13–15 的差异集中在以下几点（源自官方 Upgrade to v16 指南）：
  1. **`middleware.ts` 改名 `proxy.ts`**，导出函数改为 `proxy`，默认 **Node.js runtime**，不再支持 edge runtime。
  2. **并行路由所有 slot 必须显式 `default.tsx`**，否则构建失败（不再是"可选兜底"）。
  3. **Async Request API 彻底移除同步访问**：`params`/`searchParams`/`cookies`/`headers`/`draftMode` 只能 `await`（15 的临时同步兼容在 16 已删除）。
  4. **Turbopack 成为默认打包器**（`next dev`/`next build`），自定义 webpack 配置会导致构建失败（用 `--webpack` 退避）。
  5. **PPR 实验配置移除** → 改用 `cacheComponents`。
  6. **导航/预取增强**：Layout 去重 + 增量预取，无需改代码。
  7. **Scroll 行为**：16 不再默认覆盖 `scroll-behavior: smooth`，需要时给 `<html>` 加 `data-scroll-behavior="smooth"`。
- 旧版 Pages Router 相关内容仅作对比，非实践基线。

---

## 一、路由模式：App Router vs Pages Router

| 维度 | App Router（`app/`） | Pages Router（`pages/`） |
|---|---|---|
| 引入版本 | Next.js 13+，新项目默认 | Next.js 12 及更早的默认 |
| 组件默认 | **Server Component**（服务端组件） | 全部是 Client Component |
| 布局 | 有原生 `layout.tsx` 嵌套布局 | 靠 `_app.tsx` + `_document.tsx`，无段级布局 |
| 数据获取 | Server Component 内直接 `async/await` | 靠 `getServerSideProps` / `getStaticProps` |
| Loading/Error | 约定式 `loading.tsx` / `error.tsx` | 手动做 |
| 动态路由 | 用目录 `[id]` | 用文件名 `[id].tsx` |
| 状态 | 官方推荐 | 兼容保留，可混用 |

> 一句话：新项目用 App Router；Pages Router 只是旧项目维护与兼容。

---

## 二、文件系统路由（约定式路由）

路由就是目录结构，文件名即 URL。

```
app/
├── layout.tsx          ← 根布局（必须有 <html><body>）
├── page.tsx            ← 对应 /
├── about/
│   └── page.tsx        ← /about
├── blog/
│   ├── page.tsx        ← /blog
│   └── [slug]/
│       └── page.tsx    ← /blog/:slug（动态段）
└── api/
    └── route.ts        ← API 路由（Route Handler）
```

关键规则：
- `page.tsx` 才生成可访问的页面，纯目录不生成 URL。
- `page` / `layout` / `template` / `loading` / `error` / `not-found` 等都是**约定文件名**（special files）。
- 目录名小写、用连字符，避免驼峰与下划线（下划线开头会被视为私有文件夹，见下文）。

### 2.1 动态路由段

| 语法 | 目录示例 | 匹配 | `params` |
|---|---|---|---|
| 静态段 | `about/` | 仅 `/about` | 无 |
| 动态段 | `blog/[slug]/` | `/blog/a`、`/blog/b` | `{ slug: "a" }` |
| 可选段 | `[slug]` 且 slug 可缺省（通过 `params` 判断） | 单段 | — |
| 全捕获 | `blog/[...slug]/` | `/blog/a/b/c` | `{ slug: ["a","b","c"] }` |
| 可选全捕获 | `[[...slug]]/` | 也匹配 `/blog` | slug 可能为 `undefined` |

```tsx
// app/blog/[slug]/page.tsx
export default async function PostPage({
  params,
}: {
  params: Promise<{ slug: string }>;
}) {
  const { slug } = await params;   // App Router 里 params 是 Promise
  return <div>文章：{slug}</div>;
}
```

> **注意**：App Router 的 `params`、`searchParams` 都是 **Promise**，组件内必须 `await`。这是从旧版 `{ params }`（同步对象）迁移时最典型的报错点。

### 2.2 路由组 Route Groups `(group)`

用括号包裹目录名，**只为组织文件，不进入 URL**。

```
app/
├── (marketing)/
│   ├── layout.tsx      ← 营销页自己的布局
│   └── about/page.tsx  ← /about（URL 无 (marketing)）
└── (shop)/
    ├── layout.tsx      ← 商城页自己的布局
    └── cart/page.tsx   ← /cart
```

用途：同一层做**多个根布局**、或给一组页面单独套布局/认证逻辑。

### 2.3 私有文件夹 `_folder`

下划线开头的目录不会被路由访问，只放内部组件/工具。

```
app/
└── dashboard/
    ├── _components/    ← 不产生 URL，仅供本段 import
    └── page.tsx
```

---

## 三、并行路由 Parallel Routes（`@slot`）

一个目录下放多个 `@槽名` 子目录，同一 URL 同时渲染多块 UI（如侧边栏 + 主内容 + 弹窗），互不阻塞。

```
app/
├── layout.tsx
└── @modal/            ← 槽 slot
│   └── page.tsx
└── page.tsx           ← 默认 @children 槽
```

- 槽就是 `@name` 目录；`layout.tsx` 里按 `props` 接收：`{ children, modal }`。
- 每块可以有自己的 `loading` / `error` / `not-found`。
- **Next.js 16 起：所有 parallel slot 都必须显式提供 `default.tsx`**，否则构建直接失败。可调用 `notFound()` 或返回 `null` 兜底：

```tsx
// app/@modal/default.tsx —— Next 16 必填
import { notFound } from "next/navigation";

export default function Default() {
  notFound(); // 或直接 return null
}
```

---

## 四、拦截路由 Intercepting Routes

在 URL 不变的前提下，把别处的路由"拦截"到当前布局里渲染（典型：从列表点开详情用弹窗，刷新后变成独立页）。

| 前缀 | 相对层级 |
|---|---|
| `(.)` | 同级（同一层目录） |
| `(..)` | 上一级 |
| `(..)(..)` | 上两级 |
| `(...)` | 从 `app/` 根目录 |

```
app/
├── feed/
│   └── page.tsx            ← /feed
└── photo/
    ├── [id]/page.tsx       ← /photo/1（独立页）
    └── (.)[id]/page.tsx    ← 拦截 /photo/1，在 feed 里当弹窗渲染
```

---

## 五、Layout / Template / Loading / Error / 404

> Layout 与 Template 的完整对比见本目录《layout 和template总结.md》，这里只列层级与触发条件。

### 5.1 嵌套层级（一张图）

```
route segment
├── layout.tsx        ← 常驻，跨导航保留状态
├── template.tsx      ← 每次导航重建（可选）
├── loading.tsx       ← 由 <Suspense> 包裹，数据加载中显示
├── error.tsx         ← 段内错误边界（必须 'use client'）
├── global-error.tsx  ← 全局错误（必须含 <html><body>）
├── not-found.tsx     ← 404
└── page.tsx          ← 实际内容
```

渲染顺序（从外到内）：`Layout → Template → Loading(Suspense) → Error(边界) → Page`。

### 5.2 loading（Loading UI）

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <p>加载中…</p>;
}
```
- 原理：Next 自动用 `<Suspense>` 包裹 `page`，`loading.tsx` 即 fallback。
- 只影响该段，同段 `layout` 不会因加载而卸载。

### 5.3 error / global-error

```tsx
// app/dashboard/error.tsx  —— 必须 'use client'
"use client";
export default function ErrorBoundary({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <p>出错了：{error.message}</p>
      <button onClick={() => reset()}>重试</button>
    </div>
  );
}
```
- `error.tsx` 是**客户端**组件错误边界，能拿到 `error` 和 `reset()`。
- `global-error.tsx` 包在最外层，捕获根布局错误，**必须自带 `<html><body>`**。

### 5.4 not-found（404）

- `not-found.tsx` 该段 404 页面；`not-found()` 函数可手动触发。
- 触发条件：`notFound()` 被调用、动态段 `params` 无匹配、`default.tsx` 缺失等。

---

## 六、实现路由导航的 4 种方式

| 方式 | 类型 | 适用 | 是否预取 |
|---|---|---|---|
| `<Link>` | 客户端组件 | 页面内跳转（首选） | 是（视口内自动 prefetch） |
| `useRouter()` | 客户端 Hook | 编程式跳转（提交后跳转） | 按 `prefetch` 配置 |
| `redirect()` | 服务端函数 | 数据获取后重定向、鉴权 | — |
| `history` API | 浏览器原生 | 返回/前进、替换历史 | — |

### 1. `<Link>`

```tsx
import Link from "next/link";

<Link href="/blog/hello">文章</Link>              // 直接给完整 URL 字符串
<Link href="/blog/hello" prefetch={false}>文章</Link>  // 关闭预取
<Link href={{ pathname: "/blog/[slug]", query: { slug: "hi" } }}>动态</Link>
```

> **注意**：App Router 没有旧版的 `as` prop，直接给完整 `href`；旧代码迁移要去掉 `as`。

### 2. `useRouter()`（客户端组件）

```tsx
"use client";
import { useRouter } from "next/navigation";

const router = useRouter();
router.push("/app");        // 跳转并压栈
router.replace("/login");   // 替换当前历史
router.back();              // 返回
```

> 旧版从 `next/router` 导入，App Router 必须从 **`next/navigation`** 导入。

### 3. `redirect()`（服务端函数）

```tsx
import { redirect } from "next/navigation";

// 在 Server Component / Route Handler / Server Action 中
// 注意：proxy/middleware 里不能用 redirect()，那里应返回 NextResponse.redirect
if (!isLogin) redirect("/login");
```

### 4. History API

```tsx
window.history.pushState(null, "", "/new-path");    // 压入新历史
window.history.replaceState(null, "", "/new-path"); // 替换当前历史
```
> 直接操作 history 不会触发 Next 的客户端导航（无 RSC 数据流），通常用于特殊场景，谨慎使用。

### 5. Next 16 导航/预取行为变化

- **Layout 去重**：预取多个共享同一 layout 的 URL 时，layout 只下载一次。
- **增量预取**：只预取缓存中缺失的部分，而不是整页（会有更多但更小的预取请求，属正常）。
- **Scroll 行为**：16 起**不再默认覆盖** `scroll-behavior: smooth`。想要平滑滚动时的"即跳即停"体验，需在根 `<html>` 手动加 `data-scroll-behavior="smooth"`。
- **数据刷新新 API**（Server Actions 场景）：`refresh()` 刷新客户端 router；`updateTag()` 提供"读你写"即时失效；`revalidateTag(tag, cacheLife)` 现在需第二参数。

---

## 七、路由 Hook API（`next/navigation`）

| Hook | 作用 |
|---|---|
| `useRouter()` | 编程式导航（push / replace / back / forward / prefetch） |
| `usePathname()` | 当前路径名（不含 query），如 `/blog/a` |
| `useParams()` | 当前动态段参数对象 `{ slug: "a" }` |
| `useSearchParams()` | 查询参数（`URLSearchParams`），返回只读 |
| `useSelectedLayoutSegment(s)` | 当前激活的布局段，用于导航高亮 |

```tsx
"use client";
import { usePathname, useParams, useSearchParams } from "next/navigation";

const pathname = usePathname();        // "/blog/hello"
const params = useParams();            // { slug: "hello" }
const sp = useSearchParams();          // URLSearchParams 对象
const q = sp.get("q");
```

> 注意：`useSearchParams()` 在客户端组件里用；服务端取 query 直接用 `page` 的 `searchParams` prop（也是 Promise）。

### 7.1 类型安全：`PageProps`（Next 16）

先跑 `npx next typegen` 生成全局类型，然后页面里直接用泛型，`params` / `searchParams` 全类型推导，且强制异步：

```tsx
// app/blog/[slug]/page.tsx —— Next 16
export default async function Page(props: PageProps<"/blog/[slug]">) {
  const { slug } = await props.params;       // 类型：{ slug: string }
  const query = await props.searchParams;    // 类型：{ [key: string]: string | string[] | undefined }
  return <h1>Blog: {slug}</h1>;
}
```

另有全局 `LayoutProps`、`RouteContext`（用于 Route Handler）。

### 7.2 `useSearchParams` 详解

读取当前 URL 查询参数的客户端 Hook，返回只读的 `URLSearchParams`。

#### 核心约束

| 约束 | 说明 |
|---|---|
| 来源 | 从 `next/navigation` 导入 |
| 使用位置 | **仅客户端组件**（必须 `"use client"`） |
| 返回值 | `URLSearchParams`（**只读、不可变**） |
| 响应性 | URL 的 query 变化时，使用它的组件会重新渲染 |
| 服务端 | 不要用，服务端用 `page` 的 `searchParams` prop |

#### 读取方法（URLSearchParams 全可用）

```tsx
"use client";
import { useSearchParams } from "next/navigation";

const sp = useSearchParams();

sp.get("q");          // string | null —— 取第一个值，无则 null
sp.get("q") ?? "";    // 拿不到给默认
sp.getAll("tag");     // string[] —— 取全部重复值
sp.has("sort");       // boolean —— 是否存在
sp.keys();            // 遍历 key
sp.values();          // 遍历值
sp.entries();         // 遍历 [key, value]
sp.toString();        // "q=hello&tag=a&tag=b"
sp.size;              // 参数个数（ES2023）
```

#### 读取 + 更新（搜索/筛选/分页）

```tsx
"use client";
import { useSearchParams, useRouter, usePathname } from "next/navigation";

export default function List() {
  const sp = useSearchParams();
  const router = useRouter();
  const pathname = usePathname();

  const q = sp.get("q") ?? "";              // 搜索词
  const page = Number(sp.get("page") ?? "1"); // 当前页
  const tags = sp.getAll("tag");            // 多个筛选值

  // 更新：URLSearchParams 只读，要 set 必须新建
  const updateQuery = (key: string, value: string) => {
    const params = new URLSearchParams(sp.toString());
    params.set(key, value);
    router.push(`${pathname}?${params.toString()}`);
  };

  // 删除某个参数
  const removeQuery = (key: string) => {
    const params = new URLSearchParams(sp.toString());
    params.delete(key);
    router.replace(`${pathname}?${params.toString()}`);
  };

  return (
    <button onClick={() => updateQuery("page", String(page + 1))}>
      下一页
    </button>
  );
}
```

用 `<Link>` 也能带 query（服务端/客户端均可）：

```tsx
<Link href={{ pathname: "/list", query: { q: "react", page: 1 } }}>去搜索</Link>
```

#### 与服务端 `searchParams` prop 的对比

| 维度 | `useSearchParams()` | `page` 的 `searchParams` prop |
|---|---|---|
| 使用侧 | 客户端组件 | 服务端组件（页面） |
| 类型 | `URLSearchParams` 对象 | `Promise`（Next 16 必须 `await`） |
| 读取 | `.get()` / `.getAll()` | `await props.searchParams` 得到对象 |
| 场景 | 交互式、需响应 URL 变化的 UI | SSR 数据获取、meta、初始渲染 |

#### useSearchParams 避坑速记

1. **只读**——直接 `sp.set()` 无效，更新要 `new URLSearchParams(sp.toString())` 再 set/delete，然后 `router.push`/`replace`。
2. **静态路由段要包 `<Suspense>`**——无边界使用会整页退回客户端渲染（CSR bailout），`next build` 告警：
   ```tsx
   <Suspense fallback={<p>加载中…</p>}>
     <List />
   </Suspense>
   ```
3. **不要复制进 `useState`**——导航改 query 时会不同步，应直接用 `useSearchParams` 当"受控 URL 状态"。
4. **`get()` 返回 `null` 而非空串**——判空用 `?? ""`，别用 `||`（空串会被误伤）。
5. **`get()` 只取第一个**，重复参数用 `getAll()`；`toString()` 不会丢重复项。
6. **服务端别用这个 Hook**——会报 Hooks 只能在客户端用的错；服务端读 `await props.searchParams`。
7. **自动编码**——`URLSearchParams` 自动 encode，不用手动 `encodeURIComponent`。

---

## 八、Proxy（路由守卫，Next 16 由 middleware 改名）

**Next.js 16 起，`middleware.ts` 弃用，改名 `proxy.ts`**，导出的函数也从 `middleware` 改为 `proxy`。proxy 默认 **Node.js runtime**，**不支持 edge runtime**（想用 edge 只能保留旧 `middleware.ts`）。

`proxy.ts` 放在**项目根目录**（与 `app/` 平级，或在 `src/` 下），在请求进入页面/API 前执行，适合鉴权、重定向、维护页。

```ts
// proxy.ts —— Next 16 写法
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function proxy(req: NextRequest) {
  const token = req.cookies.get("token")?.value;
  if (!token && req.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.redirect(new URL("/login", req.url));
  }
  return NextResponse.next();
}

export const config = {
  matcher: ["/dashboard/:path*"],
};
```

> 迁移要点：`mv middleware.ts proxy.ts`，并把 `export function middleware` 改成 `export function proxy`；配置 flag 也改名，如 `skipMiddlewareUrlNormalize` → `skipProxyUrlNormalize`。

---

## 九、Route Handlers（API 路由 `route.ts`）

`app/api/xxx/route.ts` 定义一个路由的 HTTP 端点，替代旧版 `pages/api`。

```ts
// app/api/hello/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  return NextResponse.json({ message: "Hello" });
}

export async function POST(req: Request) {
  const body = await req.json();
  return NextResponse.json({ received: body });
}
```

- 与 `page.tsx` 互斥：同一目录下不能同时有 `page.tsx` 和 `route.ts`。
- 支持 `GET / POST / PUT / PATCH / DELETE / HEAD / OPTIONS`，通过 `export async function 方法名` 导出。
- 可设置 `export const dynamic = "force-dynamic"` 等运行时配置。

---

## 十、避坑速记清单

1. **`params` / `searchParams` 只能异步访问**（Next 16 已彻底移除同步访问，15 的临时兼容也被删除）。直接当对象用会报错或拿不到值。
2. **Hook 只能从 `next/navigation` 导入**（App Router），别再用 `next/router`——那是 Pages Router 的。
3. **`useRouter` 只能用于客户端组件**，必须加 `"use client"`；服务端用 `redirect()`。
4. **根 `layout.tsx` 必须有 `<html>` 和 `<body>`**；`global-error.tsx` 也得自带这两个标签。
5. **`error.tsx` 必须是客户端组件**（`"use client"`），否则不会生效。
6. **同一目录不能同时存在 `page.tsx` 和 `route.ts`**，会冲突报错。
7. **`loading.tsx` 不会阻塞同段 `layout`**——layout 常驻，只有被 Suspense 包裹的 page 段会显示 loading。
8. **并行路由每个 slot 必须显式 `default.tsx`**（Next 16 起），否则**构建直接失败**，不再是可选兜底。
9. **`<Link>` 没有 `as` prop**（App Router），直接给完整 `href`；旧代码迁移要去掉。
10. **下划线开头目录是私有文件夹**，不会被路由访问——想通过 URL 访问就别用 `_`。
11. **`notFound()` 与 `redirect()` 都会中断执行**，放在 `try/catch` 里别 catch 到它们（会吞掉重定向）。
12. **Next 16 用 `proxy.ts`，不是 `middleware.ts`**；`matcher` 只匹配前缀，覆盖子路径用 `/:path*`，别漏首页根路径。**edge 运行时不能直接改名 proxy**（proxy 固定 Node runtime），要 edge 就保留 middleware。
13. **动态段 `params` 里 key 是目录名**，多段动态如 `[lang]/[slug]` 会同时给出 `{ lang, slug }`。

---

## 十一、自测题（互动式考核）

1. `app/blog/[slug]/page.tsx` 中，访问 `/blog/hello?q=1`，`params` 和 `searchParams` 分别是什么？要不要 `await`？
2. 想让某个"进入路由就重放 CSS 动画"的区块每次导航都重置，应该用 `layout.tsx` 还是 `template.tsx`？为什么？
3. `(marketing)` 目录下的 `about/page.tsx` 对应哪个 URL？
4. Next 16 里，并行路由的槽缺了 `default.tsx` 会怎样？
5. 在 Server Component 里想跳转，应该用 `useRouter`、`<Link>`、`redirect()` 中的哪一个？为什么？
6. 拦截路由 `(..)(..)` 表示拦截**往上几级**的路由？
7. 为什么 `error.tsx` 必须加 `"use client"`？
