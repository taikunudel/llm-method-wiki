# /wiki-graph

**What it does:** Builds the knowledge graph of `[[wikilinks]]` across the wiki and writes `graph/graph.json` plus an interactive `graph/graph.html` (vis.js, nodes colored by type, searchable).

**When to use it:** Any time you want to see the shape of the wiki — clusters, hubs, isolated pages. Useful after a batch of ingests to spot connections that should exist.

**Usage:** `/wiki-graph`

---

First try running: `python tools/build_graph.py --open`

If that fails (missing Python dependencies), build the graph manually per the Graph Workflow in [CLAUDE.md](../../CLAUDE.md):

1. Use Grep to find all `[[wikilinks]]` across every file in `wiki/`.
2. Build a nodes list: one node per wiki page, with `id = relative-path`, `label = title`, `type` from frontmatter.
3. Build an edges list: one edge per `[[wikilink]]`, tagged `EXTRACTED`.
4. Infer additional implicit relationships between pages not captured by wikilinks — tag these `INFERRED` with a confidence score (0.0–1.0); tag low-confidence ones `AMBIGUOUS`.
5. Write `graph/graph.json` with `{nodes, edges, built: today}`.
6. Write `graph/graph.html` as a self-contained vis.js page (nodes colored by type, edges colored by type, interactive, searchable).

After building, summarize: node count, edge count, breakdown by type, and the most-connected nodes (hubs).

Append to `wiki/log.md`: `## [YYYY-MM-DD] graph | Knowledge graph rebuilt`.
