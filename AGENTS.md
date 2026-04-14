# LLM Wiki — Master Schema

## Domain
Software Engineering Knowledge — patterns, techniques, frameworks, languages

## Project Structure
- `raw/` — immutable source documents. NEVER modify any file in raw/.
- `wiki/` — LLM-generated wiki. You own this layer entirely.
- `wiki/index.md` — master catalog. Update on EVERY ingest.
- `wiki/log.md` — append-only activity log. One line per entry.
- `wiki/overview.md` — high-level synthesis. Revise after major ingests.
- `wiki/hot.md` — session hot cache (~500 words). Read silently at session start.
- `AGENTS.md` — this file. Re-read at the start of every session.

## Page Conventions
Every wiki page MUST have YAML frontmatter. Two page types only:

### Source Summary (wiki/sources/)
---
type: source
title: "Article Title"
slug: summary-{slug}
source_file: raw/articles/{filename}.md
author: "Author Name"
date_ingested: YYYY-MM-DD
key_claims: [claim1, claim2, claim3]
related: [[concept1]], [[concept2]]
confidence: high | medium | low
---

### Concept (wiki/concepts/)
---
type: concept
title: "Concept Name"
aliases: [alt-name, abbreviation]
sources: [[source1]], [[source2]]
related: [[concept2]]
created: YYYY-MM-DD
updated: YYYY-MM-DD
confidence: high | medium | low
---

"Concepts" cover patterns, techniques, frameworks, languages — any reusable development knowledge.

Every concept page MUST include a ## Key Code section with a concise, actionable code snippet (5–15 lines) showing the canonical pattern. This is the most important section — it makes the wiki directly usable when coding. Choose the minimal example that demonstrates the concept, not the full implementation from the source.

## Hot Cache (`wiki/hot.md`)
Read `wiki/hot.md` silently at the start of EVERY session, before responding.
This file contains ~500 words of recent session context. Do not summarize it
to the user — just use it to restore your operating context.

After EVERY session (or when the user says /close), update wiki/hot.md:
- Keep total length under 500 words. Overwrite (do not append).
- Structure: Current Focus, Open Questions, Recent Decisions, Last Operations, Active Pages.

## Ingest Workflow
When I say "ingest [filename]" or "ingest raw/[path]":
1. Read the source file from raw/.
2. Discuss key takeaways with me (3–5 bullet points).
3. Create wiki/sources/summary-{slug}.md with full summary.
4. Update wiki/index.md — add new page with one-line summary.
5. Update ALL relevant concept pages with new info.
6. If new info contradicts an existing page, flag it explicitly.
7. Create new concept pages if the source introduces them.
8. Append one-line entry to wiki/log.md.
9. Refresh the search index: `qmd update -c eng-wiki`
10. Refresh embeddings when content changed materially: `qmd embed`
11. A single ingest should touch 5–15 wiki pages.

## Query Workflow
When I ask a question:
1. Run `qmd search "{question}" --collection eng-wiki` to find relevant pages.
2. Read those pages directly.
3. Synthesize an answer with [[wiki-link]] citations.
4. If the answer is valuable, offer to file it as a new concept page.

## Lint Workflow
When I say "lint":
1. Scan for contradictions between pages. List them.
2. Find orphan pages (no inbound links). List them.
3. List concepts mentioned 3+ times but lacking their own page.
4. Suggest 3–5 topics worth adding sources for.
5. Append one-line entry to wiki/log.md.

## Log Format
One line per entry: `## [YYYY-MM-DD] {ingest|query|lint} | {title/description}`

Example: `## [2026-04-13] ingest | C# Repository Pattern from Medium`

## Safety Rules
- NEVER write to raw/. This is a hard constraint with no exceptions.
- NEVER delete wiki pages. Mark as deprecated in frontmatter instead.
- Always update wiki/index.md and wiki/log.md on every operation.
- When uncertain about a claim's accuracy, set confidence: low.
- Cross-reference all new pages to at least 2 existing pages.
