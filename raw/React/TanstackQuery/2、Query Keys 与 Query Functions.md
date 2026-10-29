# Query Keys 与 Query Functions

> **版本基线**：`@tanstack/react-query` **v5.104.x**；涉及 Next.js 内容以 **Next.js 16** 为基线。
> 官方文档：<https://tanstack.com/query/v5/docs/framework/react/guides/query-keys> · <https://tanstack.com/query/v5/docs/framework/react/guides/query-functions>

## 一、Query Keys：缓存的身份证

TanStack Query 用 **查询键（Query Key）** 管理缓存。**Key 顶层必须是数组**，且应**唯一描述**该查询函数返回的数据。

```ts
// 简单 key：列表类 / 无层级数据
useQuery({ queryKey: ['todos'], queryFn: getTodos })
```

### 分层 key 的粒度

| Key 写法 | 含义 | 典型场景 |
|---|---|---|
| `['todos']` | 整个列表 | 全部待办 |
| `['todos', 5]` | 单个条目 | id=5 的待办 |
| `['todos', { status: 'done' }]` | 筛选子集 | 已完成的待办 |
| `['todos', 5, { preview: true }]` | 带视图形态 | 预览模式 |

### 确定性哈希（Deterministic Hashing）

**对象内 key 顺序不影响哈希结果**：

```ts
// 以下两个 key 相等（同一查询）
const a = ['todos', { status: 'done', page: 1 }]
const b = ['todos', { page: 1, status: 'done' }]
```

规则：
- key 由数组的「值 + 对象的可序列化属性」做**稳定哈希**。
- 所以想定位查询必须用**完全一致**的 key（含对象属性一致）。
- 建议：**对象内属性顺序统一**（用 `queryKeyFactory` 或 `queryOptions` 保证一致，见第 12 篇）。

> ⚠️ 不要在同一查询里混用不同层级的 key 去获取同一数据——会各自缓存成两份。

## 二、Query Functions：返回 Promise 的函数

`queryFn` 可以是**任意返回 Promise 的函数**。Promise 要么 resolve 数据、要么 throw 错误。

关键约定（v5）：

- 成功 resolve 的值**不能是 `undefined`** —— resolve 出 `undefined` 会被当成**失败**（"Data was expected to be `undefined`"）。
- 想存「空」作为成功结果，用 `null`。
- 必须显式抛错；`fetch` 不会对 4xx/5xx 抛错，需手动判断。

```ts
useQuery({
  queryKey: ['repo', owner],
  queryFn: async ({ queryKey }) => {
    const [, owner] = queryKey
    const res = await fetch(`https://api.github.com/repos/${owner}/repo`)
    if (!res.ok) throw new Error('请求失败')
    return res.json()
  },
})
```

### QueryFunctionContext：queryFn 收到的参数

v5 的 queryFn 会收到一个上下文对象，常用字段：

| 字段 | 类型 | 说明 |
|---|---|---|
| `queryKey` | `QueryKey` | 本次查询的 key |
| `signal` | `AbortSignal` | 用于**取消请求**（见第 7 篇） |
| `meta` | `Record<string,unknown>` | 自定义附加信息 |
| `pageParam` | `unknown` | **仅无限查询**：当前页码/游标（见第 5 篇） |
| `direction` | `'forward'\|'backward'` | **仅无限查询**：翻页方向 |
| `client` | `QueryClient` | 当前 client 实例 |

```ts
// 取消 + 读取 queryKey 的完整示例
useQuery({
  queryKey: ['repo', owner],
  queryFn: async ({ queryKey, signal }) => {
    const [, owner] = queryKey
    const res = await fetch(`https://api.github.com/repos/${owner}/repo`, { signal })
    if (!res.ok) throw new Error('请求失败')
    return res.json()
  },
})
```

## 三、最佳实践小结

1. **从具体到泛化**写 key：`['user', userId]` 优于 `['user', { id }]` 单独拼。
2. 需要**跨组件共享**同一份缓存时，key 必须完全一致。
3. `queryFn` 通过 `context` 拿参数，**不要闭包外部可变状态**去决定请求内容（可能拿到旧值）。
4. key 对象里别放**不可序列化**的值（如函数、Date 实例直接塞）——哈希会不稳定。
5. v5 只有一个对象签名：`useQuery({ queryKey, queryFn, ... })`，没有 v4 的 `useQuery(key, fn, opts)` 多参写法。

**相关官方链接**
- Query Keys：<https://tanstack.com/query/v5/docs/framework/react/guides/query-keys>
- Query Functions：<https://tanstack.com/query/v5/docs/framework/react/guides/query-functions>


---

> **上一篇**：[1、定位与基础搭建](./1%E3%80%81%E5%AE%9A%E4%BD%8D%E4%B8%8E%E5%9F%BA%E7%A1%80%E6%90%AD%E5%BB%BA.md)　｜　**下一篇**：[3、查询状态机与 useQuery](./3%E3%80%81%E6%9F%A5%E8%AF%A2%E7%8A%B6%E6%80%81%E6%9C%BA%E4%B8%8E%20useQuery.md)
