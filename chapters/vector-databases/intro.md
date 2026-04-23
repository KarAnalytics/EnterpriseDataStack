# Introduction to Vector Databases

A **vector database** stores and indexes **high-dimensional numeric vectors** — usually **embeddings** of text, images, audio, or any other content an ML model can encode — and lets you query them by **similarity** rather than by exact match. They are the storage layer behind semantic search, recommendation systems, and the retrieval step of retrieval-augmented generation (RAG) that powers most modern LLM applications.

## Why a new kind of database

Ask a relational database *"find customers whose name is 'Ada'"* — easy. Ask it *"find documents that are semantically similar to this paragraph"* — impossible without help. The paragraph doesn't have exact matches; it has *concepts*. Vector databases solve exactly this: they store a vector per item, and given a query vector they return the most similar items fast.

## Embeddings in one paragraph

An **embedding model** maps content into a fixed-length numeric vector (e.g., 384, 768, 1536, or 3072 dimensions) such that semantically similar content lands close together in the vector space. Text embeddings typically come from models like OpenAI's `text-embedding-3-small`, Google's `gemini-embedding`, or open-source models on Hugging Face. Image, audio, and multimodal embeddings work the same way with different encoders.

For the mechanics of how embeddings are produced, see Chapter 2 of the companion AI book. This chapter focuses on the storage and retrieval side.

## Similarity metrics

A vector database answers *"which stored vectors are closest to this query vector?"* using a similarity or distance metric:

- **Cosine similarity** — angle between vectors; dominant metric for text embeddings
- **Dot product** — related to cosine when vectors are normalized; used by many OpenAI-style models
- **Euclidean (L2) distance** — straight-line distance; used for image embeddings and clustering
- **Manhattan (L1) distance** — sum of absolute differences; used occasionally

Make sure the metric you query with matches the metric the embedding model was trained for.

## Approximate Nearest Neighbor (ANN) indexes

Exact nearest-neighbor search over millions of 1536-dim vectors is slow. Vector databases use **approximate** indexes that trade a tiny amount of recall for orders-of-magnitude speedup:

- **HNSW** (Hierarchical Navigable Small World graphs) — most popular; great quality/speed trade-off
- **IVF** (Inverted File with product quantization) — partition the space into cells, search only the nearest cells; compresses vectors aggressively for low memory
- **ScaNN** — Google's state-of-the-art; used in Vertex AI Vector Search
- **FAISS** — the library Meta released that started the ANN revolution; used inside many products

You don't usually pick the index manually — but you *do* pick `ef_construction`, `ef_search`, `M`, or `nprobe` knobs that control the recall / latency trade-off.

## Filtering — the killer feature

Real applications need similarity search *within a subset* — *"documents similar to X that belong to tenant Y and were published after January 2025"*. This is called **metadata filtering** or **hybrid search**. Two strategies:

- **Pre-filter** — restrict to matching rows first, then run ANN. Correct but can be slow if the filter is narrow.
- **Post-filter** — run ANN over everything, then filter the top-k. Fast but can return too few results.
- **Filtered HNSW** — modern indexes (Pinecone, Weaviate, Qdrant) integrate filtering into the graph traversal.

## Hybrid search

For text, **dense retrieval** (embeddings) is great at semantics but can miss exact keywords. **Sparse retrieval** (BM25, TF-IDF) catches exact keyword matches. **Hybrid search** combines both scores and re-ranks — the standard approach in production RAG.

## The product landscape

- **Pinecone** — managed, fully hosted; the earliest mover
- **Chroma** — open-source; dead simple, great for prototypes
- **Weaviate** — open-source or managed; hybrid search, module ecosystem
- **Qdrant** — open-source or managed; Rust; excellent filtering
- **Milvus** — open-source; distributed; industrial scale
- **LanceDB** — open-source; embedded, works directly on object storage
- **pgvector** — extension that adds vector search to PostgreSQL — now the default for many teams because it keeps vectors alongside relational data
- **Elasticsearch / OpenSearch** — added vector support; great for teams already running them
- **Vertex AI Vector Search**, **Azure AI Search**, **AWS OpenSearch** — cloud-native managed options

## A minimal Python example (Chroma)

```python
import chromadb

client = chromadb.PersistentClient(path="./chroma_db")
col = client.get_or_create_collection(name="policies")

# Store documents (Chroma auto-embeds with its default model)
col.add(
    ids=["p1", "p2", "p3"],
    documents=[
        "Employees must submit expense reports within 30 days.",
        "Annual performance reviews occur in January and July.",
        "Office parking requires a valid permit renewed yearly.",
    ],
    metadatas=[{"category": "finance"}, {"category": "hr"}, {"category": "facilities"}],
)

# Semantic search with filter
results = col.query(
    query_texts=["how do I expense my travel costs?"],
    n_results=2,
    where={"category": "finance"},
)
print(results["documents"])
```

## Vector databases in the enterprise

This capability moves a vector database from "nice experiment" to load-bearing infrastructure:

- **Semantic search** over internal documents, product catalogs, customer tickets
- **Retrieval-Augmented Generation (RAG)** — ground LLM answers in private enterprise data
- **Recommendation** — find similar products, similar users, similar content
- **Anomaly / duplicate detection** — items whose embeddings cluster far from everything else
- **Clustering and topic discovery** — group unstructured content by meaning

All of these were prohibitively expensive a few years ago. Vector databases made them routine.

## Learning outcomes

- Explain what an embedding is and why similarity search is useful
- Choose an appropriate similarity metric for a given embedding model
- Describe how ANN indexes achieve sub-linear retrieval at cost of small recall loss
- Use metadata filters and hybrid search correctly
- Pick a vector DB product for a given scale / cost / ops profile
- Build a simple semantic-search pipeline end to end

