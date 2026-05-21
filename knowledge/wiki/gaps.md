# Wiki Gaps

Append-only log of moments where the agent had to make a substantive
modeling, statistical, or domain decision **without** wiki support.

Use this list to prioritize what to ingest next: each entry is a concrete
signal that a paper, package doc, or concept page is missing.

**Format:** one block per gap, newest at the top.

```
## [YYYY-MM-DD] <short-topic-slug>

- **Needed:** what knowledge the agent was missing
- **Surfaced by:** the task or question that exposed the gap
- **Expected page:** suggested filename under wiki/concepts/, wiki/sources/, or wiki/examples/
- **Workaround used:** what the agent did instead (cite source — paper, docs, training memory)
```

When the gap is filled (the suggested page is ingested), mark the block
with `**Resolved:** YYYY-MM-DD → [[NewPage]]` rather than deleting it,
so the history stays auditable.

---
