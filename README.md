# 个人 LLM Wiki

这个 Obsidian 仓库已经按 Karpathy 的 LLM Wiki 思路初始化。核心不是把资料简单堆进笔记，而是让 LLM 把原始资料编译成可持续更新、互相引用、可追溯证据的 Markdown 知识页。

## 目录结构

```text
.
├── raw/                 原始资料，只增不改
│   └── <主题>/
├── wiki/                LLM 编译和维护的知识页
│   ├── <主题>/
│   ├── index.md         全局索引
│   └── log.md           只追加的操作日志
├── .agents/
│   └── skills/
│       └── karpathy-llm-wiki/
│                          完整技能规则、模板、校验脚本和示例
└── YAML/                原有笔记，保持不变
```

## 三条核心规则

1. `raw/` 是不可变证据层。网页、论文、博客、PDF 转出的 Markdown、聊天记录等都先完整保存到这里。
2. `wiki/` 是持久知识层。LLM 负责提炼、组织、消解重复、记录冲突，并维护交叉链接和索引。
3. 每条关键事实都能沿着知识页的 `Raw` 字段追溯到 `raw/` 中的原始资料。

## 日常使用

让 Codex 处理知识库时，可以直接使用下面的说法：

```text
把这篇资料加入知识库：<链接、文件路径或直接粘贴内容>
```

```text
根据我的知识库，总结一下 <主题>
```

```text
对比我的知识库里关于 <A> 和 <B> 的内容，并标明来源
```

```text
检查并修复我的知识库
```

```text
把这个回答归档到知识库：<问题或回答>
```

普通查询只阅读和回答，不修改文件；只有明确要求“加入知识库”或“归档”时才会写入 `wiki/`。

## 操作含义

- `Ingest`：抓取资料到 `raw/`，判断是新知识、更新、冲突还是无新增，然后编译到 `wiki/`。
- `Query`：先读 `wiki/index.md`，再全文搜索 `wiki/`，回答时引用相应知识页。
- `Lint`：检查索引、链接、Raw 引用和证据一致性；安全项自动修复，事实冲突只报告。

## Windows 校验命令

当前电脑没有全局 `python` 命令。可使用 Codex 自带运行时执行证据检查：

```powershell
& "$env:USERPROFILE\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe" `
  ".agents\skills\karpathy-llm-wiki\scripts\check_evidence.py" .
```

运行技能自带测试：

```powershell
& "$env:USERPROFILE\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe" `
  ".agents\skills\karpathy-llm-wiki\tests\test_check_evidence.py"
```

## 上游来源

本仓库采用社区对 Karpathy LLM Wiki 工作流的完整实现：

- 仓库：https://github.com/Astro-Han/karpathy-llm-wiki
- 原始想法：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
- 许可证：MIT，许可证原文已保存在 `.agents/skills/karpathy-llm-wiki/LICENSE`

技能说明、模板和脚本位于 `.agents/skills/karpathy-llm-wiki/`，其中规则文件保持上游格式，以保证跨工具兼容。
