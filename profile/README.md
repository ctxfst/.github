# ctxfst · Context-first documents, by design

> Traditional RAG chunking strips documents of their context.  
> ctxfst is a document standard that weaves context into the structure itself,  
> so LanceDB, LightRAG, or any retrieval system gets complete semantics on read.

## The Problem with Traditional Chunking

When you split documents by fixed character counts, critical context disappears:

| Raw Chunk | What's Missing |
|-----------|----------------|
| "profit increased by 20%" | Which company? Which year? Which business unit? |
| "the API returns a 403 error" | Which API? What endpoint? What auth method? |
| "we decided to migrate to Kubernetes" | Who is "we"? What were we migrating from? |

These orphaned chunks become noise in your vector database.

## How ctxfst Solves This

ctxfst applies two techniques **at the document format level**, before any RAG pipeline touches your content:

### 1. Contextual Embeddings (Anthropic's Contextual Retrieval)

Each chunk carries its document-level context as metadata:

```diff
- "profit increased by 20%"
+ "In Apple's 2023 Q4 earnings report, regarding the Services segment: profit increased by 20%"
```

### 2. Semantic Chunking

Instead of splitting by character count, ctxfst defines boundaries at **logical units**:
- Section headers
- Concept transitions  
- Explicit semantic markers

---

## Minimal Example

```mdx
---
title: "技能知識庫"
global_context: "這份文件整理了我在 AI、RAG 和工程實踐中的核心技能點，目標是作為 Context-first Document 的範例。"
---

<Chunk id="skill-01" semantic_tags={["AI", "Embedding"]} context="基本概念：向量與 embedding">
向量是將資料轉換成數字表示的方式，以便進行相似度計算。
</Chunk>

<Chunk id="skill-02" semantic_tags={["RAG", "Pipeline"]} context="RAG 系統流程概覽">
檢索增強生成 (RAG) 流程通常包括：
1. 資料 ingestion
2. Chunking
3. Embedding
4. 向量檢索
5. 將檢索結果餵給 LLM
</Chunk>
```

**Key elements:**
- `global_context` in YAML front matter — document-level context inherited by all chunks
- `<Chunk>` tags with `semantic_tags` and `context` — explicit semantic boundaries
- Human-readable even without any pipeline — structure is the documentation

---

## What is ctxfst?

- **A cross-format specification** for context-first documents (Markdown / MDX / AsciiDoc / PDF mappings)
- **A reference compiler (`ctxc`)** that transforms context-rich documents into structured, queryable outputs
- **Forkable templates** for skills and knowledge bases, enabling teams to share a common semantic language

## Repositories

| Repo | Description |
|------|-------------|
| `ctxfst/compiler` | The `ctxc` reference implementation |
| `ctxfst/spec` | Formal specification: semantic model, fields, cross-format mappings |
| `ctxfst/ctx-skills` | Forkable skills/KB examples |

## What ctxfst is NOT

- **Not a hosted service** — it's a standard + reference implementations, not SaaS
- **Not Markdown-only** — `ctxc` processes context-first structure regardless of source format
- **Not an LLM tool** — it's a compiler; what you do with the output (embed, index, query) is up to you
- **Not a runtime** — it compiles documents, period. No APIs, no deployments.

---

## Getting Started

- **Building RAG / KB systems?** Fork `ctxfst/ctx-skills` as your team's semantic backbone
- **Implementing your own tools?** Use `ctxfst/spec` as the contract for your compilers and indexers
- **Just exploring?** Clone `ctxfst/compiler`, run a minimal example, and see context-first documents in action

---

<sub>ctxfst is in early design phase. Spec changes go through issues and RFC-style discussions. Alternative compiler implementations and real-world document examples are welcome.</sub>
