---
title: "What Is RAG?"
description: "A practical introduction to Retrieval-Augmented Generation and the main steps in a RAG workflow."
category: "AI Systems"
pubDate: 2026-07-05
tags: ["rag", "llm", "retrieval", "embeddings"]
---

**RAG**, short for **Retrieval-Augmented Generation**, is a workflow that allows a large language model (LLM) to answer questions using external knowledge sources.

An LLM can answer many questions based on the data it saw during training. However, when a user asks about information that is not contained in the model's internal knowledge, or information that has changed since training, the answer may be incomplete or inaccurate. In this situation, we can use a **RAG workflow** to build an external knowledge base that the LLM can retrieve from and use during generation.

The overall pipeline looks like this:

![RAG pipeline workflow](/images/blog/rag/rag_pipeline.png)

*Figure 1. A typical RAG workflow, divided into the indexing phase and the inference phase.*

## 1. Documents

In a RAG system, **documents** are the original knowledge sources that the system can refer to. These sources may include product manuals, PDFs, internal knowledge bases, web pages, technical reports, customer support records, or other structured and unstructured materials.

However, raw documents usually cannot be passed directly to an LLM. They may come in different formats, contain uneven information density, and include much more content than is needed for a specific question. In most cases, answering a user query requires only a small portion of the original document rather than the entire file.

This is why the next step is **chunking**.

## 2. Chunking

During **chunking**, documents are split into smaller text units called **chunks**. Each chunk should be relatively self-contained and should express a meaningful unit of information.

In practice, chunking is often based on paragraphs, headings, sections, or semantic structure. A good chunking strategy usually includes some **overlap** between adjacent chunks. Overlap means that two neighboring chunks share part of the same text. For example, chunk 1 may contain paragraphs 1-5, while chunk 2 may contain paragraphs 5-9.

This overlap reduces information loss caused by cutting text at the wrong boundary. Without overlap, an important definition, condition, or reference might be separated from the sentence that depends on it.

## 3. Embedding

After chunking, the system performs **embedding**, one of the core steps in RAG.

An **embedding model** converts each text chunk into a vector, which is a list of numbers. This is necessary because machines do not directly understand natural language in the same way humans do. By mapping text into vectors, the system can compare pieces of text mathematically.

Each chunk is assigned one embedding vector. Semantically similar chunks should have similar vectors. For example, two sentences that discuss the same product feature should be close to each other in vector space, even if they use different wording.

It is important that the user's query is later embedded using the **same embedding model**. Otherwise, the document vectors and query vector may not be comparable in a consistent semantic space.

## 4. Vector Database

A **vector database** stores and retrieves embedding vectors efficiently. It is the search layer of a RAG system.

A vector database usually stores four types of information:

1. the original text of each chunk;
2. the embedding vector corresponding to each chunk;
3. source information, such as file name, page number, section, or URL;
4. metadata, which can be used for filtering, access control, version management, and citation.

A typical record in a vector database may look like this:

```yaml
chunk_id: chunk_001
text: "F1 and F2 are designed for 150g batches..."
embedding: [0.12, -0.08, 0.35, ..., 0.21]
metadata:
  source: "Roma-X Manual"
  page: 12
  section: "Automatic Roasting Profiles"
  product: "Roma-X"
```

Once the vector database has been built, the RAG system is ready to retrieve relevant knowledge when a user asks a question.

## 5. User Query and Query Embedding

When a user submits a **query**, the system first converts that query into an embedding vector using the same embedding model that was used for the document chunks.

For example, if the user asks:

> How many grams of coffee beans should I use for Roma-X F1 mode?

The system turns this question into a **query embedding**. This query vector can then be compared with the vectors stored in the vector database.

## 6. Retrieval

**Retrieval** is the process of searching the vector database for chunks that are most similar to the query embedding.

The system takes the query embedding and compares it against the stored chunk embeddings. It then returns the most relevant chunks. This process is often called **top-k retrieval**, where `k` is the number of results returned. For example, the system may retrieve the top 5 most relevant chunks.

To improve retrieval accuracy, many RAG systems also use **reranking**. Reranking means that another model reorders the retrieved results and removes chunks that are not sufficiently relevant.

A reranker is often implemented with a **cross-encoder** or an LLM-based judge. Unlike standard embedding retrieval, which encodes the query and chunk separately, a cross-encoder reads the query and the chunk together and then outputs a relevance score. This usually improves precision, but it is slower than direct vector search.

Engineering rules can also be added to reranking. This is called **rule-based reranking**. For example, newer documents may be ranked ahead of older documents, or official manuals may be ranked ahead of user comments.

## 7. Prompt Construction

After the system finds relevant chunks, it performs **prompt construction**. This step inserts the retrieved context into the prompt that will be sent to the LLM.

A simple prompt may look like this:

```text
You are a helpful assistant. Answer the user's question based only on the provided context.

Context:
[1] F1 and F2 are designed for 150g batches.
[2] F3 to F5 are designed for 300g batches.
[3] Manual mode supports 100g to 300g.

User Question:
How many grams of coffee beans should I use for Roma-X F1 mode?

Instructions:
Answer clearly and cite the relevant context.
```

This prompt gives the LLM both the user's question and the supporting evidence needed to answer it. The instruction also constrains the model to answer based on the provided context instead of relying only on its internal knowledge.

## 8. LLM Generation

During **LLM generation**, the model reads the constructed prompt and produces the final answer.

Because the prompt includes retrieved context, the LLM can generate a more grounded and accurate response. In the example above, the model should answer that Roma-X F1 mode is designed for a **150g batch**, and it should cite the relevant context if citation is required.

## Summary

RAG does not require retraining the LLM. It is primarily an engineering workflow that connects an LLM with an external knowledge base.

The effectiveness of a RAG system depends mainly on two things:

1. whether the system can accurately embed both documents and user queries;
2. whether it can retrieve the right information from the vector database quickly and reliably.

If these two parts are well designed, RAG can help an LLM generate answers based on external, up-to-date, and domain-specific knowledge.
