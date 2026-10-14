# Next.js Layout vs Template

> App Router 中 `layout.tsx` 与 `template.tsx` 的用法、异同与避坑。

## 一、是什么

### Layout（布局）

`layout.tsx` 是路由段的**共享 UI**，在嵌套路由之间持久存在。

```
app/
├── layout.tsx          ← 根布局（必须有，含 <html><body>）
├── page.tsx
└── dashboard/
    ├── layout.tsx      ← dashboard 段共享布局
    ├── page.tsx
    └── settings/
        └── page.tsx
```

访问 `/dashboard` → `/dashboard/settings` 时，根 layout 和 dashboard layout **都不会重新挂载**，只切换中间的 `page`。

### Template（模板）

`template.tsx` 与 layout 结构几乎一样，但**每次导航都会重新挂载**。

```
app/
└── dashboard/
    ├── template.tsx   ← 每次进入 dashboard 段都新建实例
    ├── layout.tsx
    └── page.tsx
```

## 二、相同点

| 维度 | 说明 |
|---|---|
| 文件位置 | 都放在路由段目录下，文件名不同（`layout.tsx` / `template.tsx`） |
| 接收 children | 都接收 `children: React.ReactNode`，包裹子路由 |
| 可嵌套 | 都可以多级嵌套，形成层级 |
| 可 fetch 数据 | 都能在组件内直接 async/await 取数据 |
| 默认 Server Component | 不加 `'use client'` 时都是 Server Component |
| 可加 `'use client'` | 需要交互状态时都能声明为 Client Component |
| 路由参数 | 都能拿到 `params`（动态路由段） |

## 三、核心区别

| 维度 | Layout | Template |
|---|---|---|
| **导航时行为** | 持久存在，不重新挂载 | 每次导航都重新挂载（新实例） |
| **State** | 保留——组件状态、滚动位置都在 | 重置——全部重新初始化 |
| **CSS 动画** | 只在首次挂载播放一次 | 每次导航重新播放 |
| **DOM 节点** | 复用同一个 DOM 节点 | 创建新的 DOM 节点 |
| **Hydration** | 只 hydrate 一次 | 每次导航重新 hydrate |
| **性能** | 更轻量（不重复执行） | 有额外开销（重复挂载/hydrate） |
| **使用场景** | 导航栏、侧边栏、面包屑——需要常驻 UI | 需要每次进入都重置状态或重放动画的 UI |

### 一张图理解

```
Layout:  [Layout ──────────────────────────────]
         [        Page A        →   Page B    ]   ← Layout 不换

Template: [Template A ──────]
          [Page A]
          ↓ 导航
          [Template B ──────]                     ← 整个 Template 重建
          [Page B]
```

## 四、代码示例

### Layout

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <Sidebar />   {/* 导航时不卸载，状态保留 */}
      <main>{children}</main>
    </div>
  );
}
```

### Template

```tsx
// app/dashboard/template.tsx
export default function DashboardTemplate({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="animate-fade-in">  {/* 每次导航都重新播放动画 */}
      {children}
    </div>
  );
}
```

## 五、避坑速记清单

1. **根 layout 必须包含 `<html>` 和 `<body>`**，不能漏，否则报错。
2. **想在导航时播放进入动画？用 template，不是 layout。** Layout 的 CSS animation 只跑一次，第二次导航不会重放——这是最常见的踩坑。
3. **Layout 里的 state 会跨路由保留**——如果你在 layout 里放了一个 `useState` 的展开/收起面板，切路由后它还是展开的。不希望这样就移到 page 里或用 template。
4. **Template 会多一层 DOM 包裹**，如果你的 CSS 选择器依赖直接父子关系（如 `> .child`），要算上这层。
5. **Template 每次都重新 fetch 数据**——如果数据不常变，用 layout；template 的 fetch 每次导航都跑。
6. **Layout 不能向 children 传响应式 props**——layout 是 Server Component 时没有 React state；客户端 layout 传的 props 也不会因导航更新。需要传值用 Context 或 URL params。
7. **不要在 layout 里做每次路由都需要重置的表单/向导**——state 会残留，用 template 或拆到 page。
8. **template.tsx 不替代 layout.tsx**——两者可以同时存在，嵌套顺序是 `Layout > Template > Page`。
9. **Parallel Routes / Intercepting Routes 下**，layout 的持久化行为可能导致你以为"页面没刷新"——实际是 layout 复用了，page 确实换了。
10. **性能敏感场景慎用 template**——每次导航重新挂载 + 重新 hydrate，高频切换路由时体感卡顿。
