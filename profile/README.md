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
| Hard to update context without re-embedding | Update context independently |
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
  - id: project:payment-gateway
    tags: [Project, FinTech, Go]
    context: "Payment system handling 10k TPS with hybrid architecture"
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

### Why This Matters for Modern RAG

| System | How ctxfst Helps |
|--------|------------------|
| **LanceDB** | Store `context`, `content`, `tags` as separate columns; filter by tags, embed context+content |
| **LightRAG** | `tags` become graph nodes; `context` improves entity extraction |
| **HippoRAG 2** | Structured `id` enables cross-document linking; tags form knowledge graph edges |
| **LlamaIndex Hybrid** | Metadata filters (`tags`) + vector search (`context+content`) |
| **RAG-Anything** | Multi-modal attributes in frontmatter (timestamps, sources) |
| **DyG-RAG** | Dynamic graphs built from evolving tag relationships |

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

**ctxfst gives you the same embedding quality as Anthropic's approach** (context + content combined for embedding), **plus structured metadata** for:
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

## Export to Modern RAG Systems

ctxfst documents export to JSON ready for ingestion:

```bash
python export_to_lancedb.py document.md --output chunks.json
```

Output:
```json
{
  "id": "skill:python",
  "context": "Author's Python skills...",
  "content": "## Python\nI use Python for...",
  "tags": ["Python", "Backend"],
  "source": "skills.md"
}
```

### LanceDB Ingestion Example

```python
import lancedb

db = lancedb.connect("./db")
table = db.create_table("chunks", data)

# Hybrid query: vector + tag filter
results = table.search(query_embedding)
    .where("'Python' IN tags")
    .limit(10)
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

<sub>ctxfst is in early design phase. Contributions and real-world examples welcome.</sub>
