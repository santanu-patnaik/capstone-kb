---
doc_id: arch_ptn_008
pattern_name: RAG (Retrieval-Augmented Generation) with Vector Database
category: agentic_systems
typical_team_size: 2–6
complexity_fit: low_to_medium
---

# RAG with Vector Database

## What it is
An LLM-based system whose outputs are grounded in a corpus of retrieved documents. Documents are chunked, embedded, and stored in a vector database. At query time, the query is embedded, the top-k most similar chunks are retrieved (optionally filtered by metadata), and the chunks are injected into the LLM prompt as context. The LLM generates a response grounded in the retrieved chunks, with citations.

## When to pick it
- Your LLM needs to answer from a specific corpus (org docs, product manuals, past artifacts) rather than general world knowledge.
- The corpus is too large to fit in a prompt but too small to justify fine-tuning.
- You need citations / traceability — every claim should point to a source.
- Freshness matters — the corpus can be updated without retraining a model.

## When not to pick it
- The knowledge is general enough that the base LLM already handles it well.
- The corpus is tiny (few hundred tokens) — just stuff it in the prompt.
- Relationships between documents matter more than similarity (graph RAG might fit better).

## Key design choices
- **Chunk size.** 500–1000 tokens is a common default. Too small loses context; too large wastes the context window and dilutes relevance.
- **Chunking strategy.** Structural (by heading / section) is almost always better than naive fixed-window when documents have structure.
- **Embedding model.** OpenAI text-embedding-3-small is cheap and strong for English. sentence-transformers (MiniLM, mpnet) are free and good enough for many use cases. Match the embedding model across ingest and query.
- **Vector DB.** Chroma/FAISS for local; Pinecone/Qdrant/Weaviate for managed. Pinecone is strong for metadata filtering and namespaces; Qdrant is strong for hybrid search out of the box.
- **Metadata.** Tag every chunk with source, section, date, and domain. Filter before you retrieve — cheaper than retrieving everything and then filtering.
- **top_k.** Start at 5. Too high dilutes; too low misses relevant chunks. Measure with a retrieval eval set.
- **Similarity threshold.** 0.70–0.80 cosine is a typical floor for "actually relevant." Below the floor, skip the chunk rather than stuffing marginal matches.
- **Reranking.** For higher precision, rerank top-20 retrieved with a cross-encoder (Cohere Rerank, bge-reranker).

## Trade-offs
- **Pro:** Fresh, auditable, grounded outputs. No fine-tuning infrastructure needed.
- **Pro:** Small corpus changes don't require model changes.
- **Con:** Retrieval quality is the ceiling on output quality — bad retrieval = bad grounding.
- **Con:** Chunking choices matter a lot and are rarely revisited; get it right up front.
- **Con:** Hallucination still possible if the LLM ignores the context; enforce grounding with evals.

## Typical stack fit
Pinecone/Qdrant/Chroma for vector store. OpenAI/Voyage/Cohere/sentence-transformers for embeddings. LangChain or LlamaIndex for ingest and retrieval pipelines. Any LLM for generation.

## Related patterns
- **Hybrid search:** Combine vector similarity with keyword (BM25) for better recall on rare terms.
- **GraphRAG:** When relationships between entities matter more than similarity.
- **Agentic RAG:** The retrieval step is itself a decision point — the agent can choose to retrieve or not, and can reformulate queries.
