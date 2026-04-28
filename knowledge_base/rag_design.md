---
doc_id: rag_design_v1
title: RAG Design — Chunking, Metadata, Retrieval Strategy
version: 1.0
owner: capstone author
last_updated: 2026-04-20
---

# RAG Design — Chunking Strategy, Metadata Schema, and Retrieval Parameters

Companion to the knowledge base. Explains how the six source types are ingested, indexed, and retrieved. Every non-trivial choice below is justified — evaluators should be able to trace each choice back to a reason.

## 1. Embedding model and index

- **Embedding model:** OpenAI `text-embedding-3-small` (1536 dimensions). Chosen over `text-embedding-3-large` because small is ~80% of the quality at ~20% of the cost for English technical text, and the retrieval corpus is small enough (<1000 chunks total) that marginal recall gains don't justify 5x cost. Same model used at ingest and query time (critical — mismatched models break retrieval).
- **Vector DB:** Pinecone serverless, single index with one namespace per source type. Serverless because the corpus is small and query volume is bursty (per-BRD, not constant); pay-per-query economics fit the workload.
- **Index config:** dimension=1536, metric=cosine, serverless on AWS us-east-1.

## 2. Namespace design

One Pinecone namespace per source type. Namespaces are used instead of metadata-only filtering because:
- They are a first-class query dimension in Pinecone and cheaper than metadata filters for source-type scoping.
- Each agent retrieves from a deliberately narrow set of namespaces, which is cleaner to reason about than a global index with filters.

| Namespace | Source type | Expected chunk count |
|---|---|---|
| `past_brds` | Past BRDs | 50–100 |
| `plan_templates` | Plan templates | 30–50 |
| `architecture_patterns` | Architecture patterns | 40–60 |
| `project_history` | Project history rows | 10–20 |
| `org_standards` | Org engineering standards | 25–40 |
| `tech_stack_decisions` | Tech stack decision log | 20–40 |

Total expected index: roughly 200–350 chunks. Well within Pinecone serverless free-tier limits for the capstone.

## 3. Per-source chunking strategy

Chunk size is not one-size-fits-all. Each source type has a different structure, and chunking is tuned per type.

### 3.1 Past BRDs
- **Strategy:** Structural — split on H2 headings (`## Functional Requirements`, `## Non-Functional Requirements`, etc.).
- **Why:** BRD sections have semantic boundaries. Fixed-window chunks would put functional and non-functional requirements in the same chunk, which hurts retrieval when the question is about one or the other.
- **Target size:** 600–900 tokens per chunk; if a section is longer, sub-split on bullet/paragraph boundary.
- **Overlap:** 50 tokens at section boundaries to preserve context for cross-section claims.
- **Typical BRD:** 8–15 chunks depending on complexity.

### 3.2 Plan templates
- **Strategy:** Structural — one chunk per phase (e.g., "Phase 1 — Discovery") plus one chunk for "Standard risks", one for "Milestones", one for "Team composition".
- **Why:** Phases are the natural unit of reuse; a planner agent retrieving context for "Phase 2 of a medium project" should get Phase 2 content from similar templates, not an entire template.
- **Target size:** 300–600 tokens per chunk.
- **Overlap:** None; phases are self-contained.

### 3.3 Architecture patterns
- **Strategy:** One chunk per pattern document. Patterns are already short (400–700 tokens each) and have a standard structure (What / When to pick / When not / Trade-offs).
- **Why:** A pattern is the atomic unit. Splitting "When to pick" away from "When not to pick" would produce half-truths in retrieval.
- **Target size:** 400–700 tokens; patterns longer than 800 tokens get sub-split on H2.
- **Overlap:** None.

### 3.4 Project history
- **Strategy:** One chunk per project row. Each row is 150–250 tokens and self-contained.
- **Why:** Historical analogs are retrieved as individual rows; a Schedule Estimator asking "how did past medium-complexity Guidewire projects perform" wants rows, not a table summary.
- **Target size:** 150–300 tokens per chunk.
- **Overlap:** None.

### 3.5 Org standards
- **Strategy:** Structural — split on H2 headings (approved stacks, CI/CD, observability, security, architecture review criteria, etc.). Sub-split long sections on H3.
- **Why:** An agent asking about approved data stores shouldn't get back a chunk dominated by CI/CD content.
- **Target size:** 400–800 tokens per chunk.
- **Overlap:** None; sections are self-contained.

### 3.6 Tech stack decisions
- **Strategy:** One chunk per ADR (one per TSD-### entry).
- **Why:** Each decision is a self-contained unit and is retrieved as such. Splitting an ADR would risk losing the rationale or outcome from the decision.
- **Target size:** 150–350 tokens per chunk.
- **Overlap:** None.

## 4. Metadata schema

Every chunk is tagged with a consistent metadata envelope. Filtering on metadata before vector similarity is both cheaper and more precise.

```json
{
  "doc_id": "brd_002",
  "source_type": "past_brd",
  "domain": "insurance_pc",
  "complexity": "medium",
  "year": 2024,
  "section": "non_functional_requirements",
  "tags": ["Guidewire", "regulated", "ML_ops"],
  "chunk_index": 3,
  "token_count": 742
}
```

| Field | Type | Applies to | Purpose |
|---|---|---|---|
| `doc_id` | string | all | Traceability back to source document |
| `source_type` | enum | all | Already implicit in namespace; duplicated here for cross-namespace queries |
| `domain` | enum | BRDs, history, ADRs | Filter by industry vertical (insurance_pc, insurance_life, healthcare, itsm, etc.) |
| `complexity` | enum | BRDs, templates, history | `low` / `medium` / `high` — match new BRD complexity to historical analogs |
| `year` | int | BRDs, history, ADRs | Prefer recent context; bias toward last 3 years |
| `section` | string | BRDs, standards | Which H2 section the chunk came from |
| `tags` | string[] | all | Free-form — `Guidewire`, `HIPAA`, `Okta`, `microservices`, etc. |
| `chunk_index` | int | all | Position within the source doc; used for adjacent-chunk retrieval if needed |
| `token_count` | int | all | Observability and cost accounting |
| `pattern_name` | string | architecture_patterns | e.g., `modular_monolith`, `event_driven` |
| `team_size_historical` | int | history | Allow filter on team size ranges |
| `applicability` | string | templates | Short text describing which projects this template fits |

## 5. Retrieval parameters

### 5.1 Default retrieval config

| Parameter | Default | Rationale |
|---|---|---|
| `top_k` | 5 | Empirically hits the sweet spot for dense technical text — above 7, marginal chunks dilute the prompt; below 3, relevant chunks get missed. |
| `similarity_threshold` | 0.75 (cosine) | Below this, chunks are noise more often than signal for this corpus. Measured on a small held-out retrieval eval set. |
| `include_metadata` | true | Agents cite chunks by `doc_id` + `chunk_index`. |
| `include_values` | false | Saves payload size. |

### 5.2 Per-agent retrieval profiles

Each agent queries a specific combination of namespaces and filters. This is deliberate — the Tech Stack Recommender should not be retrieving plan templates.

| Agent | Namespaces queried | Typical metadata filter |
|---|---|---|
| Engineering Plan Generator | `past_brds`, `plan_templates`, `project_history` | `complexity` matches BRD complexity; `domain` matches or is null |
| Schedule Estimator | `project_history`, `plan_templates` | `complexity` matches; `team_size_historical` within ±50% of proposed team |
| Solution Architect | `architecture_patterns`, `past_brds`, `org_standards` | `domain` matches; `section = "non_functional_requirements"` for NFR-driven retrieval |
| PoC Planner | `past_brds` (POC-tagged), `project_history` (POC-status) | `tags` contains `POC`; or `status = "POC complete"` |
| Tech Stack Recommender | `tech_stack_decisions`, `org_standards`, `project_history` | `domain` matches; year within last 3 years |
| Critic | All namespaces | No filter — the Critic needs to verify claims against the whole corpus |

### 5.3 Query formulation

Each specialist agent issues 1–3 queries per request, not one blob query. Example for the Solution Architect receiving a BRD for a P&C claims system:

1. `"P&C claims integration Guidewire architecture pattern"` → architecture_patterns + past_brds
2. `"NFR latency availability claims processing"` → past_brds (section filter: non_functional_requirements)
3. `"approved messaging platform event-driven"` → org_standards

Multi-query is essential because a single embedding of the whole BRD is a poor retrieval key — it dilutes specific needs into a generic vector.

## 6. Ingest pipeline

Implemented as an n8n workflow (mirrors the overall tool track):

1. **Watch** — new file dropped into the knowledge base folder triggers ingest.
2. **Parse** — front-matter extraction + markdown body.
3. **Chunk** — per-source chunker (from Section 3).
4. **Embed** — OpenAI `text-embedding-3-small` via the org LLM gateway.
5. **Upsert** — into the correct Pinecone namespace with the metadata envelope (Section 4).
6. **Log** — every chunk's `doc_id`, `chunk_index`, `token_count`, embedding latency, upsert status.

Re-ingest on file change is destructive per-doc (delete all chunks with `doc_id = X`, re-insert) rather than incremental. The corpus is small enough that this is simpler and safer than tracking incremental diffs.

## 7. Evaluation hooks

- **Retrieval eval set:** a handful of (query, expected_doc_ids) pairs per namespace. Run after any chunking or parameter change. Track recall@5 and MRR.
- **Grounding eval:** in the end-to-end pipeline, measure what fraction of claim-bearing sentences in generated artifacts cite a retrieved chunk. Target ≥75% per the brief.
- **Retrieval cost:** log embedding tokens and Pinecone query count per BRD; target is under $0.05 per end-to-end BRD run.

## 8. Known limitations and future work

- **No hybrid search yet.** Pure vector. For rare terms (product names, regulation codes) BM25 would help; Phase 2 candidate.
- **No reranking.** A cross-encoder reranker (Cohere Rerank or bge-reranker) over the top-20 would improve precision; adds cost and latency, evaluate after baseline.
- **English only.** Corpus is English; multilingual would require a different embedding model and re-ingest.
- **Static corpus.** No continuous ingest from live systems (e.g., current projects); the corpus is curated and versioned manually. For production, a live pipeline from Confluence / Jira / Guidewire would be the next step.
