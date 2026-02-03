# ctxfst · Context-first documents, by design

> Traditional RAG chunking strips documents of their context.  
> ctxfst is a document standard that weaves context into the structure itself,  
> so LanceDB, LightRAG, or any retrieval system gets complete semantics on read.

---

## Why not Anthropic's Original Approach?

Anthropic's [Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval) is groundbreaking, but its original implementation has limitations:

| Anthropic Original | ctxfst Approach |
|--------------------|-----------------|
| Context **merged into** chunk content | Context stored as **structured metadata** |
| Single combined string for embedding | Separate `context`, `content`, `tags` columns |
| Hard to update context without re-embedding | Update context without rewriting content (re-embed depends on strategy) |
| One-dimensional retrieval | Multi-dimensional: vector + graph + filter |

### The Problem with "Context Prepending"

```python
# Anthropic's approach: merge context + content into one string
stored_chunk = "This is about Q2 revenue... " + "Revenue grew 3%"
embedding = embed(stored_chunk)
```

This works for simple vector search, but **modern RAG systems need structure**:
- **LanceDB** — needs separate columns for filtering and hybrid search
- **LightRAG / HippoRAG** — extract entities from `tags`, build knowledge graphs
- **LlamaIndex** — hybrid retrieval with metadata filters
- **RAG-Anything / DyG-RAG** — multi-modal and dynamic graph retrieval

---

## ctxfst: Structured Frontmatter Format

ctxfst separates **metadata** from **content** using YAML frontmatter:

```markdown
---
chunks:
  - id: skill:python
    tags: [Python, Backend, FastAPI]
    context: "Author's Python skills for REST APIs and data pipelines"
    created_at: "2026-02-03"
    version: 1
    priority: high
    dependencies: []
  - id: project:payment-gateway
    tags: [Project, FinTech, Go]
    context: "Payment system handling 10k TPS with hybrid architecture"
    created_at: "2026-01-15"
    version: 2
    type: text
---

<Chunk id="skill:python">
## Python
I use Python for building REST APIs and data pipelines...
</Chunk>

<Chunk id="project:payment-gateway">
## Payment Gateway
Built a payment processing system handling 10k transactions per second...
</Chunk>
```

### Why This Matters for Modern RAG (2026)

| System | How ctxfst Helps |
|--------|------------------|
| **LanceDB** | Store `context`, `content`, `tags` as separate columns; filter by tags, embed context+content |
| **LightRAG** | `tags` become graph nodes; `dependencies` create edges; `context` improves entity extraction |
| **HippoRAG 2** | Structured `id` enables cross-document linking; tags form knowledge graph edges |
| **LlamaIndex Agentic** | `priority` hints guide agent retrieval order; hybrid search with metadata filters |
| **RAG-Anything** | Multi-modal support via `type` field (text/image/video/audio) |
| **DyG-RAG / T-GRAG** | Temporal graphs from `created_at` + `version`; dynamic relationships via tags |
| **LangGraph** | `dependencies` enable prerequisite context loading; `priority` for retrieval planning |

---

## How ctxfst Differs from Anthropic

```
┌─────────────────────────────────────────────────────────────┐
│  Anthropic Original                                         │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ "Context here... + Original content all merged together"││
│  └─────────────────────────────────────────────────────────┘│
│                         ↓                                   │
│                 Single embedding                            │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│  ctxfst Frontmatter Format                                  │
│  ┌──────────────┐ ┌───────────────┐ ┌─────────────────────┐ │
│  │ context      │ │ tags          │ │ content             │ │
│  │ (metadata)   │ │ (filterable)  │ │ (original text)     │ │
│  └──────────────┘ └───────────────┘ └─────────────────────┘ │
│         ↓                 ↓                   ↓             │
│    Graph nodes      Filter queries     Vector embedding     │
│    Entity extraction                   (context + content)  │
└─────────────────────────────────────────────────────────────┘
```

ctxfst supports embedding either `content` only or `context + content` (same quality as Anthropic's approach), **plus structured metadata** for:
- Hybrid search (vector + BM25 + filter)
- Knowledge graph construction
- Incremental updates
- Multi-system compatibility

---

## Semantic Chunking + Structured Metadata

ctxfst combines two techniques:

### 1. Semantic Chunking (Upstream)

Split documents by **meaning boundaries**, not character count:

```
Fixed chunking:    [500 chars] [500 chars] [500 chars]  ← may cut mid-sentence
Semantic chunking: [Topic A]   [Topic B]   [Topic C]    ← respects meaning
```

### 2. Structured Frontmatter (Downstream)

Store chunk metadata in YAML for programmatic access:

```yaml
chunks:
  - id: skill:python
    tags: [Python, Backend]
    context: "50-100 token description..."
```

This combination enables **end-to-end context preservation** from document authoring to retrieval.

---

## 2026 RAG Extensions

ctxfst evolves with RAG trends. The format supports optional extension fields for advanced use cases:

### Temporal RAG

```yaml
chunks:
  - id: skill:llm-prompt
    created_at: "2026-02-03"  # ISO date for temporal indexing
    version: 3                 # Track content updates over time
```

Enables **point-in-time retrieval** (query knowledge as of specific date) and **version control** for evolving knowledge bases. Compatible with T-GRAG, Temporal GraphRAG, and versioned vector stores.

### Agentic RAG

```yaml
chunks:
  - id: api:auth
    priority: high             # Hints for agent retrieval ordering
    dependencies: [api:setup]  # Prerequisite knowledge for agents
```

Helps AI agents understand **which chunks to retrieve first** and **what context is needed** before answering. Used by LangGraph, AutoGen RAG, and LlamaIndex agent pipelines.

### Multi-Modal RAG

```yaml
chunks:
  - id: diagram:architecture
    type: image                # text, image, video, audio
    content_path: "./arch.png" # Path to media file
```

Enables structured retrieval of **non-text content** alongside text. Prepares documents for CLIP-based retrieval, Milvus RAG-Anything, and multi-modal LightRAG.

### Extension Timeline

| Extension | Purpose | Since | Spec |
|-----------|---------|-------|------|
| **Temporal RAG** | `created_at`, `version` for time-aware queries | v1.1 (2026-02) | Optional |
| **Agentic RAG** | `priority`, `dependencies` for agent guidance | v1.1 (2026-02) | Optional |
| **Multi-Modal RAG** | `type`, `content_path` for media content | v1.1 (2026-02) | Optional |

All extension fields are **optional** and **backward compatible**. Documents without them work with all existing RAG systems.

---

## Export to Modern RAG Systems

ctxfst documents export to JSON ready for ingestion:

```bash
python3 skill-chunk-md/scripts/export_to_lancedb.py document.md --output chunks.json
```

Output (with 2026 extensions):
```json
{
  "id": "skill:python",
  "context": "Author's Python skills...",
  "content": "## Python\nI use Python for...",
  "tags": ["Python", "Backend"],
  "created_at": "2026-02-03",
  "version": 1,
  "type": "text",
  "priority": "high",
  "dependencies": [],
  "source": "skills.md"
}
```

### LanceDB Ingestion Example

```python
import json
import lancedb

db = lancedb.connect("./db")
with open("chunks.json", "r", encoding="utf-8") as f:
    data = json.load(f)

table = db.create_table("chunks", data)

# Hybrid query: vector + tag filter
results = table.search(query_embedding).where("'Python' IN tags").limit(10)
```

### LightRAG Integration

```python
# Tags become graph nodes automatically
# Each chunk links to its semantic tags
# Cross-document relationships emerge from shared tags
```

---

## Repositories

| Repo | Description |
|------|-------------|
| [`skill-chunk-md`](https://github.com/ctxfst/skill-chunk-md) | Markdown → ctxfst converter with validation and export scripts |
| `ctxfst/compiler` | The `ctxc` reference implementation (coming soon) |
| `ctxfst/spec` | Formal specification (coming soon) |

---

## Evolution Roadmap

ctxfst is designed to adapt to RAG advances while maintaining backward compatibility:

### Released

- ✅ **v1.0** (2026-01) — Core frontmatter format with `context`, `tags`, `content` separation
- ✅ **v1.1** (2026-02) — Temporal, Agentic, Multi-Modal extensions for 2026 RAG trends

### In Progress

- 🚧 **v1.2** (2026-Q2) — Parametric RAG metadata support, streaming chunk updates
- 🚧 **Integration examples** — Reference implementations for LanceDB, LightRAG, LlamaIndex

### Planned

- 📋 **v2.0** (2026-Q3) — Self-learning embeddings, feedback loop metadata
- 📋 **ctxc compiler** — Automatic context generation from source documents
- 📋 **Formal spec** — JSON Schema validation and cross-language parsers

**Philosophy**: ctxfst evolves as RAG systems evolve, but all extensions are **optional** and **backward compatible**. Simple use cases stay simple; advanced features are available when needed.

---

## Getting Started

1. **Fork [`skill-chunk-md`](https://github.com/ctxfst/skill-chunk-md)** — includes validation + export scripts
2. **Convert your documents** — add frontmatter with chunk definitions
3. **Export to your RAG system** — JSON output ready for LanceDB/LightRAG

---

## References

### Anthropic Contextual Retrieval
- [Research Blog](https://www.anthropic.com/research/contextual-retrieval)
- [Engineering Post](https://www.anthropic.com/engineering/contextual-retrieval)
- [Cookbook Guide](https://platform.claude.com/cookbook/capabilities-contextual-embeddings-guide)

### Semantic Chunking
- [NAACL 2025 Paper](https://aclanthology.org/2025.findings-naacl.114/)
- [Weaviate Chunking Strategies](https://weaviate.io/blog/chunking-strategies-for-rag)

### Modern RAG Frameworks
- [LanceDB](https://lancedb.github.io/lancedb/)
- [LightRAG](https://github.com/HKUDS/LightRAG)
- [HippoRAG 2](https://github.com/OSU-NLP-Group/HippoRAG)
- [LlamaIndex Hybrid Search](https://docs.llamaindex.ai/)

---

<sub>ctxfst evolves with RAG trends. Contributions, real-world examples, and extension proposals welcome. 🚀</sub>
