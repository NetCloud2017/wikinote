# Next.js MDX 知识点

> MDX = Markdown + JSX 的超集，允许在 `.mdx` 文件里直接写 JSX、`import` React 组件。涵盖本地 MDX 渲染（`@next/mdx`，App Router）、组件注入与样式、Frontmatter、remark/rehype 插件、远程加载 MDX（`next-mdx-remote` / `next-mdx-remote-client`）、缓存与安全注意事项，附避坑速记清单与自测题。

## 版本说明

- **本文以 Next.js 16 为基线**（App Router）。所有代码示例按 16 的写法。
- MDX 在 App Router 中的官方入口是 `app/guides/mdx`；本地内容用官方 `@next/mdx`，远程内容官方推荐社区包 `next-mdx-remote-client`（`next-mdx-remote` 为另一主流选择）。
- Next 16 默认打包器为 **Turbopack**，它对 remark/rehype 插件有额外约束（见第五节）。
- `mdxRs`（Rust 编译器）仍是**实验性**功能，官方不建议用于生产。

---

## 一、什么是 MDX

MDX 是 Markdown 的**超集**：既保留 markdown 语法，又允许直接写 JSX 并导入 React 组件。

- 示例：

  ```mdx
  I **love** using <a href="https://nextjs.org/">Next.js</a>
  ```

  编译输出：

  ```html
  <p>I <strong>love</strong> using <a href="https://nextjs.org/">Next.js</a></p>
  ```

- 典型场景：博客、文档、营销落地页等"正文里要嵌可交互组件"的内容型页面。
- Next.js 支持两类 MDX 内容：
  | 类型 | 来源 | 渲染时机 | 官方推荐方案 |
  |---|---|---|---|
  | 本地 MDX | 项目内 `.mdx` 文件 | 构建时编译为 React 组件 | `@next/mdx` |
  | 远程 MDX | CMS / 数据库 / 远程接口动态 fetch | 运行时在服务端获取并渲染 | `next-mdx-remote-client`（或 `next-mdx-remote`） |

> 底层原理：React 并不理解 markdown。markdown 需先经 `remark` 解析为 AST → `rehype` 转 HTML AST → 生成 HTML。用 `@next/mdx` 时这一串已封装好，无需手动用 `unified`/`remark-parse`/`rehype-*` 拼装。

---

## 二、本地 MDX：`@next/mdx`（官方）

### 2.1 安装依赖

```bash
npm install @next/mdx @mdx-js/loader @mdx-js/react @types/mdx
```

### 2.2 配置 `next.config.mjs`

在项目根目录把 MDX 配置合并进 Next 配置，并让 `.md` / `.mdx` 参与页面编译：

```mjs
import createMDX from '@next/mdx'

/** @type {import('next').NextConfig} */
const nextConfig = {
  // 把 markdown / MDX 加进可编译为页面的扩展名
  pageExtensions: ['js', 'jsx', 'md', 'mdx', 'ts', 'tsx'],
  // 其他 Next.js 配置……
}

const withMDX = createMDX({
  // 需要的 markdown 插件写在这里（见第五节）
  options: { remarkPlugins: [], rehypePlugins: [] },
})

export default withMDX(nextConfig)
```

- 默认**只编译 `.mdx`**。要一并处理 `.md`，给 `createMDX` 加 `extension`：

  ```mjs
  const withMDX = createMDX({
    extension: /\.(md|mdx)$/,
  })
  ```

### 2.3 必须有 `mdx-components.tsx`（App Router 强制项）

在项目根目录（与 `app` / `pages` 同级，或 `src` 内）创建 `mdx-components.tsx`。**在 App Router 中使用 `@next/mdx` 时，这个文件是必需的，没有它 MDX 无法工作。**

```tsx
import type { MDXComponents } from 'mdx/types'

const components: MDXComponents = {}

export function useMDXComponents(): MDXComponents {
  return components
}
```

- 必须具名导出 `useMDXComponents`（不接受参数）。
- 空对象也能让 MDX 跑起来；自定义样式/组件也集中在这里（见第三节）。

### 2.4 三种渲染方式

**① 文件系统路由**：`.mdx` 文件直接当页面。

```
my-project/
├── app/
│   └── mdx-page/
│       └── page.mdx        ← 对应 /mdx-page
├── mdx-components.tsx
└── package.json
```

`app/mdx-page/page.mdx`：

```mdx
import { MyComponent } from 'my-component'

# Welcome to my MDX page!

这是 **加粗** 和 _斜体_，以及 markdown 列表：
- One
- Two
- Three

下面是 React 组件：
<MyComponent />
```

**② 普通导入**：MDX 文件放在别处，用 `import` 当组件用：

```tsx
import Welcome from '@/markdown/welcome.mdx'

export default function Page() {
  return <Welcome />
}
```

**③ 动态导入（强烈建议加 `.mdx` 扩展名）**：适合"一堆文章按 slug 取一篇"：

```
app/blog/[slug]/page.tsx
content/welcome.mdx、about.mdx
```

```tsx
export default async function Page({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params
  const { default: Post } = await import(`@/content/${slug}.mdx`)
  return <Post />
}

export function generateStaticParams() {
  return [{ slug: 'welcome' }, { slug: 'about' }]
}

// 只允许上面预渲染过的 slug，其余返回 404
export const dynamicParams = false
```

> 注意：`params` 在 Next 16 里是 `Promise`，必须 `await`；动态 import 路径中**必须写明 `.mdx` 扩展名**，否则解析不到。

---

## 三、组件注入与样式

Markdown 渲染时会映射到原生 HTML 元素（`## heading` → `<h2>`，列表 → `<ul><li>`）。可把这些元素替换成自定义组件，作用范围分三档。

### 3.1 全局（影响所有 MDX 文件）

在 `mdx-components.tsx` 里覆盖默认元素，例如把 `img` 换成 `next/image`：

```tsx
import type { MDXComponents } from 'mdx/types'
import Image, { ImageProps } from 'next/image'

const components = {
  h1: ({ children }) => <h1 style={{ color: 'red', fontSize: '48px' }}>{children}</h1>,
  img: (props) => (
    <Image
      sizes="100vw"
      style={{ width: '100%', height: 'auto' }}
      {...(props as ImageProps)}
    />
  ),
} satisfies MDXComponents

export function useMDXComponents(): MDXComponents {
  return components
}
```

### 3.2 局部（覆盖并合并全局）

通过 `components` prop 传给导入的 MDX 组件，会与全局组件合并、局部优先：

```tsx
import Welcome from '@/markdown/welcome.mdx'

function CustomH1({ children }) {
  return <h1 style={{ color: 'blue', fontSize: '100px' }}>{children}</h1>
}

const overrideComponents = { h1: CustomH1 }

export default function Page() {
  return <Welcome components={overrideComponents} />
}
```

### 3.3 共享布局

App Router 自带嵌套布局，`app/mdx-page/layout.tsx` 会在该 MDX 页外围套一层共享结构/样式。

### 3.4 Tailwind + typography

用 Tailwind 时配合 `@tailwindcss/typography` 的 `prose` 类给 markdown 内容排版（标题字号、行距等），配合共享布局使用。

---

## 四、Frontmatter

**`@next/mdx` 默认不解析 frontmatter（YAML 头）**。可选方案：

- `remark-frontmatter`
- `remark-mdx-frontmatter`
- `gray-matter`（在读取文件时解析）

不过 `@next/mdx` **支持 JS 导出**，可把元数据写成具名导出，在 MDX 文件外引用：

```mdx
export const metadata = {
  author: 'John Doe',
}

# Blog post
```

```tsx
import BlogPost, { metadata } from '@/content/blog-post.mdx'

export default function Page() {
  console.log('metadata: ', metadata) // => { author: 'John Doe' }
  return <BlogPost />
}
```

常见用法：配合 Node 的 `fs` / `globby` 遍历文章目录，提取各篇元数据建博客索引页（注意 `fs`/`globby` 只能在服务端使用）。

---

## 五、remark / rehype 插件

用于扩展 markdown 能力，例如 `remark-gfm` 支持 GitHub 风格表格/删除线：

```mjs
import remarkGfm from 'remark-gfm'
import createMDX from '@next/mdx'

const nextConfig = {
  pageExtensions: ['js', 'jsx', 'md', 'mdx', 'ts', 'tsx'],
}

const withMDX = createMDX({
  options: {
    remarkPlugins: [remarkGfm],
    rehypePlugins: [],
  },
})

export default withMDX(nextConfig)
```

使用注意事项（Next 16 尤其重要）：
- **remark/rehype 生态只支持 ESM**，所以配置文件必须用 `next.config.mjs` 或 `next.config.ts`，不能用 `.js`。
- **Turbopack 下插件名要写字符串**，且**带不可序列化选项的插件暂不支持**（JS 函数无法传给 Rust）：

  ```mjs
  const withMDX = createMDX({
    options: {
      remarkPlugins: [
        'remark-gfm',                    // 无选项：写字符串
        ['remark-toc', { heading: 'The Table' }], // 带选项：数组
      ],
      rehypePlugins: [
        'rehype-slug',
        ['rehype-katex', { strict: true, throwOnError: true }],
      ],
    },
  })
  ```

---

## 六、远程加载 MDX（重点）

当 MDX 内容不在项目里、而是存在 CMS / 数据库 / 远程接口时，可在**服务端动态 fetch** 再渲染。官方文档推荐的社区包是 `next-mdx-remote-client`；另一主流选择是 Hashicorp 的 `next-mdx-remote`（v6，React 19 请用 `next-mdx-remote-client` v2）。

### 6.1 最小用法（RSC）

```tsx
import { MDXRemote } from 'next-mdx-remote-client/rsc'

export default async function RemoteMdxPage() {
  // MDX 文本——来自数据库、CMS、fetch 任意来源
  const res = await fetch('https://...')
  const markdown = await res.text()

  return <MDXRemote source={markdown} />
}
```

- `MDXRemote`（RSC 版）是**异步 Server Component**，完全在服务端编译并流式渲染，MDX 编译代码**不会打进客户端 bundle**。
- `next-mdx-remote` 对应写法：`import { MDXRemote } from 'next-mdx-remote/rsc'`。

### 6.2 提取 frontmatter / 传参：`compileMDX`

需要拿到 frontmatter 或向 MDX 注入作用域变量时用 `compileMDX`（同样来自 `/rsc`）：

```tsx
import { compileMDX } from 'next-mdx-remote/rsc'

const { content, frontmatter } = await compileMDX({
  source: markdown,
  // scope: { someVar: 42 },   // 可注入变量给 MDX 内的表达式用
  options: {
    remarkPlugins: [remarkGfm],
    rehypePlugins: [],
  },
})
```

### 6.3 缓存策略

`MDXRemote` 组件本身只负责"渲染给定文本"，**获取源数据的 fetch 由你自己控制缓存**。常用手段：

```tsx
const res = await fetch('https://cms.example/posts/1', {
  // 增量静态再生成：每 3600 秒后台刷新一次
  next: { revalidate: 3600 },
})
```

- 想长期缓存且允许按需失效，可用 `next: { tags: [...] }` 配合 `revalidateTag()`。
- 不想缓存（实时数据）就给 `cache: 'no-store'` 或 `next: { revalidate: 0 }`。

### 6.4 安全警告（RCE）

> **MDX 会编译为 JavaScript 并在服务端执行。** 若内容来自不可信来源，等于远程代码执行（RCE）。**只能从你信任的源获取 MDX**——不要直接渲染用户输入 / 第三方未审核内容。

---

## 七、MDX 与 Server / Client 边界（使用注意事项）

- App Router 中，`.mdx` 文件默认被编译为 **Server Component**。
- **交互组件（用 `useState` / `useEffect` / 事件）不要在 MDX 文件里直接写**，应单独建一个 `'use client'` 文件，再 `import` 进 MDX：

  ```mdx
  import { Counter } from '@/components/counter' // counter.tsx 顶部有 'use client'

  # 计数演示

  <Counter />
  ```

- 远程渲染的 `MDXRemote` 是 async Server Component，同理：给它的交互子组件要各自是 `'use client'` 文件。
- 若 `components` prop 来自一个 `'use client'` 模块，作为 RSC 的 prop 传入时要格外注意——客户端模块的组件 map 不能被当作纯服务端对象直接传给 RSC 渲染（可序列化约束）。一般把组件映射放在服务端侧的普通模块导出。

---

## 八、避坑速记清单

1. **App Router 没建 `mdx-components.tsx` → MDX 完全无法工作**，必须建且要具名导出 `useMDXComponents`。
2. **`@next/mdx` 默认不解析 frontmatter**，别指望 YAML 头自动变成数据；要解析就上 `remark-frontmatter` / `gray-matter`，或用 `export const metadata = {}`。
3. **动态 `import` 必须写 `.mdx` 扩展名**（如 `@/content/${slug}.mdx`），漏了会解析不到。
4. **remark/rehype 插件只支持 ESM** → 配置文件必须是 `next.config.mjs` / `next.config.ts`；**Turbopack 下插件名写字符串**，带函数/不可序列化选项的插件暂不支持。
5. **远程 MDX = 服务端执行 JS，有 RCE 风险**，只从受信任源获取，绝不渲染用户提交的未审核内容。
6. **交互组件要包在 `'use client'` 文件里再 import**，别在 `.mdx` 里写 state/effect。
7. **`.md` 默认不编译**，要处理需给 `createMDX` 加 `extension: /\.(md|mdx)$/`。
8. **`mdxRs`（Rust 编译器）是实验特性**，别在生产用；需要稳定就保持默认 JS 编译器。
9. **Next 16 动态路由 `params` 是 `Promise`，必须 `await`**（MDX 动态页同理）。
10. **远程缓存由 fetch 控制**，`MDXRemote` 不管源数据缓存；用 `next: { revalidate }` / `tags` 自己配。

---

## 九、自测题（互动式考核）

1. App Router 里跑 `@next/mdx`，为什么必须建 `mdx-components.tsx`？它必须导出什么函数？
2. 在 `app/blog/[slug]/page.tsx` 里 `await import(\`@/content/${slug}.mdx\`)`，漏写 `.mdx` 扩展名会怎样？
3. `@next/mdx` 支持 frontmatter 吗？要实现博客文章头部数据，有哪些正确做法？
4. 想把远程 CMS 的 MDX 渲染出来，官方文档推荐哪个包？`MDXRemote` 的 RSC 用法长什么样？
5. 为什么说"远程 MDX 有 RCE 风险"？渲染前要遵守什么底线？
6. 想在 MDX 页面里放一个带 `useState` 的计数器，正确姿势是什么？为什么不能直接在 `.mdx` 里写？
7. Next 16 默认用 Turbopack，配置 `remark-gfm` 插件时要注意什么（文件名、插件写法）？
8. 远程 MDX 想每 1 小时后台刷新、不阻塞首屏，缓存怎么配？
