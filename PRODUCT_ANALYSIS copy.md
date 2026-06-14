# AuraNovaBrain — Product Analysis

> Generated: 2026-06-14  
> Scope: Full codebase, architecture, and content analysis

---

## 1. Architecture

AuraNovaBrain is an **LLM-maintained knowledge base** (LLM Wiki) — not a traditional software application, but a structured, version-controlled collection of interlinked markdown files. The architecture is defined by `CLAUDE.md`.

### Three-Layer Design

```
┌──────────────────────────────────────┐
│  CLAUDE.md — Schema & Rules          │  ← LLM behavior definition
├──────────────────────────────────────┤
│  raw/ — Immutable Source Documents   │  ← Original articles, PDFs, data
├──────────────────────────────────────┤
│  wiki/ — LLM-Owned Knowledge Base    │  ← Synthesized, cross-referenced
│    ├── entities/    (14 pages)       │
│    ├── concepts/    (13 pages)       │
│    ├── sources/     (10 pages)       │
│    ├── synthesis/   (1 page)         │
│    ├── comparisons/ (0 pages)        │
│    ├── index.md                      │
│    └── log.md                        │
└──────────────────────────────────────┘
```

| Layer | Role | Mutable? |
|-------|------|----------|
| `CLAUDE.md` | Tells the LLM **how to behave** | Co-evolves with human |
| `raw/` | Immutable source documents (PDFs, articles) | ❌ Never modified |
| `wiki/` | LLM-authored knowledge — entities, concepts, sources, synthesis | ✅ LLM writes/maintains |

### Inheritance from the LLM Wiki Pattern

The system follows a compound-knowledge model: each source ingested increases the value of all existing pages through cross-referencing. Value compounds with every operation.

---

## 2. Domain & Purpose

**Domain:** Dermatological public health + AI-driven skincare recommendation systems.

**Thesis (from synthesis page):** The most impactful intersection of AI and dermatological public health lies in **dual-use systems** — tools that serve individual consumers today (personalized skincare) while building infrastructure that can be redirected toward public health goals tomorrow (population surveillance, primary care triage, health promotion).

### Knowledge Coverage

| Dimension | Content |
|-----------|---------|
| **Policy** | WHO WHA78.15 (2025), advocacy briefs, policy evolution 2017→2025 |
| **Epidemiology** | GBD 2010–2021: 4.86B cases, 42.9M DALYs, fungal/bacterial dominance |
| **AI/ML** | Fuzzy logic → Transformers → Foundation models (PRISM/UNI 92.5% accuracy) |
| **Clinical** | 13 skin diseases (acne through vitiligo), pathophysiology, comorbidities |
| **Patient voice** | PatientsLikeMe data, stigma analysis, community engagement patterns |
| **Workforce** | Dermatologist shortage: 0–3/million in Sub-Saharan Africa |

---

## 3. Key Functions (Workflows)

Defined in `CLAUDE.md` as LLM agent workflows:

### 3.1 Ingest (Source Processing)

```
Read source → Discuss takeaways → Create source page
  → Update entity pages → Update concept pages
    → Cross-reference → Update index → Append to log
```

**Processing depth per source:**
- Bibliographic metadata (author, date, DOI, venue)
- 3–5 key takeaways
- Data extraction (tables, statistics, quotes)
- Entity/concept extraction
- Cross-references to existing wiki pages
- Contradiction detection

### 3.2 Query (Question Answering)

```
Read index → Find relevant pages → Synthesize answer with citations
  → Offer to file as synthesis/comparison page
```

### 3.3 Lint (Health Check)

```
Contradiction detection → Staleness check → Orphan detection
  → Missing page suggestion → Broken link check → Report
```

### 3.4 Update (Revision After New Information)

```
Read affected pages + new source → Rewrite to integrate (not append)
  → Update log → Update frontmatter dates
```

---

## 4. Technologies & Patterns

### Core Technologies

| Technology | Role |
|------------|------|
| **Markdown + YAML frontmatter** | Page format — machine-readable metadata + human-readable content |
| **Obsidian wikilinks** (`[[page-name]]`) | Cross-referencing — Obsidian-compatible graph of knowledge |
| **Git** | Version control — every change is tracked, revertible, auditable |
| **LLM Agent (Claude)** | Wiki maintainer — ingests sources, synthesizes, cross-references, lints |
| **`.obsidian/`** | Obsidian workspace config — local editor compatibility |

### AI/ML Technologies Referenced in Wiki

| Technology | Source | Application |
|------------|--------|-------------|
| **Fuzzy Logic** | Hemantha et al. (2022) | Skin type → product matching with membership functions |
| **NLP Sentiment Analysis** | Hemantha et al. (2022) | Feature-level review polarity extraction |
| **Transformer Encoder** | Lee et al. (2024) | Ingredient sequence → 18 cosmetic effect predictions |
| **U-Net** | Lee et al. (2024) | Facial image → 6 skin concern grades |
| **Foundation Models** (PRISM, UNI, Prov-GigaPath) | Song et al. (2025) | Off-the-shelf skin cancer diagnosis (92.5% accuracy) |

### Design Patterns

| Pattern | Implementation |
|---------|---------------|
| **Immutable/Mutable separation** | `raw/` (read-only) vs `wiki/` (LLM-authored) |
| **Schema-as-code** | `CLAUDE.md` defines structure, conventions, workflows |
| **Append-only log** | `wiki/log.md` — chronological audit trail of all operations |
| **Compound knowledge** | Each new source enriches all linked pages; value grows non-linearly |
| **Frontmatter metadata** | Every page: `type`, `tags`, `created`, `updated`, `sources[]`, `status` |
| **Contradiction tracking** | `> ⚠️ Contradiction:` blockquotes + frontmatter `contradictions: []` |
| **Inline citations** | `[src](sources/source-slug.md)` for provenance |
| **Master index** | `wiki/index.md` — full catalog with one-line summaries |

---

## 5. Unique Achievements

### 5.1 LLM Wiki Schema Invention
The `CLAUDE.md` file is itself a **meta-document**: a domain-specific language for LLM behavior. It defines not just file structure but entire workflows (ingest, query, lint, update) with step-by-step specificity. This is a novel pattern — the schema *is* the agent's prompt.

### 5.2 Cross-Domain Synthesis
The project bridges three normally disconnected domains:

```
Computer Science (AI/ML)  ←→  Public Health Policy (WHO)  ←→  Epidemiology (GBD)
```

The synthesis page `ai-global-dermatological-health.md` demonstrates this convergence with:
- Side-by-side comparison table (WHO resolution vs AI paper)
- 4 explicit convergence points
- Gap analysis (5 identified gaps)
- Policy origin story (6-year timeline)
- AI trajectory (3-generation evolution)

### 5.3 Dual-Use Thesis
The core intellectual contribution: consumer AI skincare tools can serve as **public health infrastructure**. This is not an abstract claim — it's backed by concrete examples:
- WHO Skin NTD App (NTDs + 24 common diseases) as precedent
- Foundation models (92.5% accuracy) deployable "off-the-shelf" without local retraining
- Ingredient safety analysis as preventive public health intervention

### 5.4 Source Processing Depth
10 sources ingested → 42 wiki pages, each with:
- Structured YAML frontmatter
- Cross-references to related pages
- Data tables extracted from academic papers
- Connection to the broader synthesis thesis
- Contradiction flags where sources disagree

### 5.5 Patient Voice Gap Discovery
The Szeto et al. (2024) analysis revealed a counterintuitive insight documented in the wiki: patient communities reflect **morbidity** (DALYs), not **prevalence**. Psoriasis (6,451 PLM users, 40.8M cases) dominates online communities while fungal diseases (578M cases, 290 PLM users) are invisible — a signal that AI tools targeting acne/cosmetics may be addressing the wrong problem.

---

## 6. Interesting Solutions

### 6.1 The Wiki as Codebase
> "The wiki is the codebase. You are the programmer." — CLAUDE.md

This reframes knowledge management as software engineering. Every wiki page is a "module" with imports (wikilinks), exports (tags + index entries), and version history (git). The LLM is the "programmer" applying software engineering practices to knowledge.

### 6.2 Contradiction Detection System
When sources disagree, the system doesn't hide it — it surfaces disagreements explicitly:
- `> ⚠️ Contradiction:` blockquotes on both affected pages
- `contradictions: ["other-page.md"]` in frontmatter
- Example: Hemantha (2022) emphasizes review sentiment → Lee (2024) argues ingredient sequence is sufficient and reviews are unreliable. The wiki flags this, notes they may be complementary, and links both pages.

### 6.3 Append-Only Audit Log
`wiki/log.md` records every operation with a consistent format:
```
[YYYY-MM-DD] type | description
```
This provides full provenance — you can trace when any page was created, from which source, and in which ingest round.

### 6.4 YAML Frontmatter as Semantic Layer
Every page carries machine-readable metadata:
- `type`: entity | concept | source | synthesis | comparison
- `status`: draft | reviewed | stable
- `sources[]`: traceable provenance for every claim
- `tags[]`: cross-cutting categories

This enables programmatic operations — lint passes can find pages by type/status, query workflows can filter by sources, etc.

### 6.5 Obsidian + Git Compatibility
The entire system is designed to work seamlessly with **Obsidian** (local markdown editor with graph visualization) while remaining **Git-friendly** (plain text files, no databases). This means:
- Visual knowledge graph in Obsidian
- Version history in Git
- No vendor lock-in
- Portable across any markdown editor

### 6.6 Inline Citation System
Claims are traceable to sources via `[src](sources/source-slug.md)` — a lightweight citation system that mirrors academic referencing but in a format native to markdown wikis.

### 6.7 Self-Documenting Architecture
The `CLAUDE.md` schema serves dual purpose:
1. **Runtime:** the LLM's behavioral instructions
2. **Documentation:** any human reading it understands the entire system

No separate architecture document needed — the schema *is* the documentation.

---

## 7. Statistics

| Metric | Value |
|--------|-------|
| Total wiki pages | 42 |
| Sources ingested | 10 |
| Entity pages | 14 (13 diseases + WHO) |
| Concept pages | 13 |
| Source pages | 10 |
| Synthesis pages | 1 |
| Comparison pages | 0 |
| Raw documents | 22 (20 markdown + 2 PDFs) |
| Ingest rounds | 3 |
| Cross-references (wikilinks) | ~150+ |
| AI generations tracked | Fuzzy logic (2022) → Transformers (2024) → Foundation models (2025) |
| GBD studies covered | GBD 2010, 2013, 2019, 2021 |
| Policy timeline | 2017–2025 (8 years) |
| Skin diseases documented | 13 (acne through vitiligo) |

---

## 8. Gaps & Opportunities

1. **No comparison pages yet** — the `wiki/comparisons/` directory is empty. Rich opportunities exist for side-by-side analyses (Hemantha vs Lee, GBD vs PLM, etc.)
2. **No stable pages** — all 42 pages are status `draft`. None have been promoted to `reviewed` or `stable`.
3. **Validation gap** — the wiki acknowledges that no source addresses regulatory validation requirements for AI-based skin assessment.
4. **No code/implementation** — the wiki references AI architectures but contains no executable code. A future direction could be implementing a prototype recommendation system based on the documented architectures.
5. **Patient voice integration** — the PLM data analysis reveals that AI tools target acne/cosmetics, but psoriasis has 7× more community engagement. The wiki flags this gap but hasn't proposed a solution.
