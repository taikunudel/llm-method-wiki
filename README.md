# llm-method-wiki

A method-aware fork of [SamurAIGPT/llm-wiki-agent](https://github.com/SamurAIGPT/llm-wiki-agent) (which itself derives from [nashsu/llm_wiki](https://github.com/nashsu/llm_wiki)) — same "drop sources in, get a wiki" idea, but the templates capture **operational** knowledge (canonical API calls, paper-recommended hyperparameters, argument quirks, silent-failure modes, runnable code examples) instead of just summaries. Built so a coding agent can actually *use* the wiki when writing model code, not merely read it for context.

The shipped wiki seeds the knowledge base with 8 auto-insurance / Tweedie modeling papers and 8 runnable R snippets — useful as a starting corpus or just as a worked example of what the schema produces.

## What's different from upstream

- **Method-paper template** — every source page tagged `method`/`software` has required sections: `Canonical API`, `Key Hyperparameters`, `Argument Quirks`, `Failure Modes`, `Code Example`, `Domain Pitfalls`.
- **Concept-page template variants** — `Method/Software Concept` vs `Domain Concept`, each with its own required sections (originals were free-form).
- **`wiki/examples/` directory** — one runnable snippet per method, written to compile end-to-end against a canonical dataset. Agents are encouraged to copy verbatim, then modify.
- **`wiki-naive/`** — the pre-regen state preserved side-by-side so you can `diff` what the method-aware schema produced versus the naive schema on the *same* sources.
- **Three-agent compatibility** — `CLAUDE.md`, `AGENTS.md`, `GEMINI.md` carry identical schema content, so Claude Code, Codex/OpenCode, and Gemini CLI all read the same rules.
- **Domain-specific source templates** — Diary/Journal and Meeting Notes, alongside the generic source template.
- **9 standalone Python tools** in `tools/`: `build_graph.py`, `file_to_md.py`, `heal.py`, `health.py`, `ingest.py`, `lint.py`, `pdf2md.py`, `query.py`, `refresh.py`.

Original upstream README preserved as [`README-UPSTREAM.md`](README-UPSTREAM.md).

---

## Setup — using this with openclaw (or any agent that reads `AGENTS.md`)

**Step 1.** Clone this repo into your openclaw workspace so the wiki sits next to your work:

```bash
cd ~/.openclaw/workspace          # or wherever your agent's workspace lives
git clone https://github.com/taikunudel/llm-method-wiki llm-wiki-agent
```

(The folder name `llm-wiki-agent` is what the schema and the snippet below expect. If you prefer another name, change every `llm-wiki-agent/` path in the snippet to match.)

**Step 2.** Tell your agent the wiki exists by pasting this block into your workspace's `AGENTS.md` (and/or `CLAUDE.md`, `GEMINI.md` — whichever your harness loads). Drop it anywhere near the top:

```markdown
## 📚 Knowledge Base — Optional

There's a local wiki at `llm-wiki-agent/wiki/` that may have relevant
background for your task. Layout:

- `index.md`   — one-line catalog of every page (start here)
- `overview.md` — current synthesis across all sources
- `sources/`   — per-document summaries
- `concepts/`  — ideas, methods, frameworks
- `entities/`  — people, packages, organizations
- `examples/`  — runnable snippets, if any exist for the topic

**Why a glance can be worth it:** the wiki tends to capture
package-specific gotchas, paper-recommended hyperparameters, and silent-
failure modes that often aren't in your training data. When it's
relevant, skipping it can cost more debugging time than reading it
would.

**Cost is low:** the wiki is small — `index.md` is typically under 100
lines. Scanning the catalog takes seconds.

Use it when useful, ignore it otherwise. Contents change over time, so
don't assume what's in there — check `index.md` if you think it might
help. If you do use a page, a `[[PageName]]` reference in code comments
or in your trajectory makes the work easier to retrace.
```

That's it. The next session your agent loads `AGENTS.md`, it will know the wiki is there, where to find the catalog, and how to cite pages back. The block is *advisory* — the agent decides whether to consult it based on task relevance.

**Optional Step 3 — trajectory logging for audit:** if you want to later audit *whether* the agent used the wiki, also paste the `## 📋 Task Trajectory — Log What You Do` section below into the same `AGENTS.md`. It tells the agent to write JSON events to `audit/trajectories/<task-id>.jsonl` as it works. See `audit/` design notes (not in this repo yet — a future addition).

<details>
<summary>Trajectory block (optional, click to expand)</summary>

```markdown
## 📋 Task Trajectory — Log What You Do

When working on a modeling, coding, or domain task that consults the wiki at
`llm-wiki-agent/wiki/`, record what you do as you do it so the work can be
audited later.

**Where:** `audit/trajectories/<task-id>.jsonl` at the workspace root. Create
the `audit/trajectories/` folder if it doesn't exist. `<task-id>` is a short
slug — e.g. `auto-ins-2026-05-16` or whatever uniquely identifies this run.

**Format:** one JSON object per line, appended in real time (not batched at
the end — timestamps matter to the auditor). Required event types:

| event | required fields |
|---|---|
| `task_start` | `ts`, `task_id`, `goal`, `wiki_root`, `git_branch_start` |
| `wiki_read` | `ts`, `task_id`, `page_id`, `bytes_read`, `sha256` |
| `decision` | `ts`, `task_id`, `summary`, `cites: [page_id,...]`, `rationale` |
| `code_edit` | `ts`, `task_id`, `path`, `lines_added`, `lines_removed`, `cites: [page_id,...]` |
| `code_run` | `ts`, `task_id`, `cmd`, `exit_code`, `summary` |
| `gap_surfaced` | `ts`, `task_id`, `concept`, `expected_page` |
| `task_end` | `ts`, `task_id`, `summary`, `git_branch_end`, `git_sha_end` |

`ts` is ISO-8601 UTC (`2026-05-16T14:22:01Z`). `page_id` is the path relative
to `wiki/` (e.g. `concepts/TweedieDistribution.md`). `sha256` is the hash of
the page contents at the time you read it — the auditor recomputes from git
to detect fabricated reads.

**Rules:**
- Append-only. Never edit or delete past entries.
- Emit each event immediately after the action it describes, not in a batch.
- Every `decision` and `code_edit` MUST have a non-empty `cites` array if the
  wiki informed it. Empty cites on a substantive decision = faithfulness fail.
- Mirror every `Read` of a `wiki/**` file as a `wiki_read` event.

**Honesty:** the auditor cross-checks your trajectory against git history
(`git log -p`, file mtimes) and any harness-level tool logs. Omissions and
fabrications are detectable. Be complete — it's cheaper than getting caught.
```

</details>

---

## What's seeded in the wiki

The `wiki/` ships pre-populated with the auto-insurance / Tweedie corpus used to develop this fork:

- **8 source papers** — Smyth-Jorgensen (2002), Dunn-Smyth (2008), Frees-Meyers-Cummings (2011), Wood (2011), Zhang (2013), Qian-Yang-Zou (2016), Yang-Qian-Zou (2016), Delong-Lindholm-Wüthrich (2021).
- **8 runnable R examples** in `wiki/examples/` — one per method, each calling its package against `cplm::AutoClaim`.
- **17 concept pages** — Tweedie distribution, GLM, GAM, gradient boosting, grouped elastic net, Gini index, adverse selection, etc.
- **14 entity pages** — authors and R packages.

Delete or replace the seed content if you only want the schema; it's small (~150 KB) and easily removed with `rm -rf wiki/* raw/papers/* && git checkout wiki/index.md wiki/log.md wiki/overview.md`.

---

## License

MIT — see [`LICENSE`](LICENSE). Inherits from upstream.

## Credits

- [SamurAIGPT/llm-wiki-agent](https://github.com/SamurAIGPT/llm-wiki-agent) — direct upstream this fork descends from.
- [nashsu/llm_wiki](https://github.com/nashsu/llm_wiki) — the original Tauri desktop app that inspired the whole approach.
