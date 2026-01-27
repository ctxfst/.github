# ctxfst · Context-first documents, by design

> ctxfst defines a document standard and toolchain that treat context as a first-class citizen, so your docs stay readable for humans and precise for machines.

## What is ctxfst?

- A cross-format specification for context-first documents (e.g. Markdown / MDX / AsciiDoc / PDF mappings).
- A reference compiler (**ctxc**) that turns context-rich documents into structured, queryable outputs.
- A set of forkable examples and templates for skills / knowledge bases, so teams can share a common language.

## What we intentionally do not do

- We do not lock you into a single runtime or framework: any language can implement the ctxfst spec.
- We do not aim to be "a hosted service": ctxfst is a standard plus reference implementations, not a SaaS product.
- We do not treat resumes or personal branding as the core use case: skills repos are reference docs and templates, not portfolios.

- ### Non-goals of ctxc (the reference compiler)

To avoid confusion about scope:

- **ctxc is not Markdown-only**: it processes context-first structure, regardless of source format (Markdown, MDX, AsciiDoc, or others).
- **ctxc is not an LLM tool**: it is a compiler that produces structured output; what you do with that output (embed it, index it, feed it to LLMs) is up to you.
- **ctxc is not a runtime**: it does not execute code, serve APIs, or manage deployments. It compiles documents, period.

## Repositories in this organization

- `ctxfst/compiler` – the ctxc reference implementation that compiles context-first documents into structured data.
- `ctxfst/spec` – the formal ctxfst specification: semantic model, fields, cross-format mappings, and compatibility rules. **ctxc** is the reference compiler that implements this spec, not the format itself.
- `ctxfst/ctx-skills` – forkable skills / KB examples that demonstrate how to write ctxfst-compliant documents.

## How you can use it

- Building LLM / RAG / internal KB systems? Start by forking `ctxfst/ctx-skills` as the backbone of your team's skills and knowledge.
- Implementing your own tools? Use `ctxfst/spec` as the contract that your compilers, indexers, and platforms align to.
- Just exploring the idea? Clone `ctxfst/compiler`, run a minimal example, and see what "context-first documents" look like in practice.

---

ctxfst is currently in an early design and reference implementation phase. Changes to the spec will go through issues and RFC-style discussions. We welcome alternative implementations of the compiler in any language, as well as real-world document examples from your domain.
