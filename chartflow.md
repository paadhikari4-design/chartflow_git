# ChartFlow Architecture: Prakash Chatflow

**GitHub Repository:** [paadhikari4-design/chartflow_git](https://github.com/paadhikari4-design/chartflow_git)

This document outlines the architecture and configuration of the **Prakash Chatflow**, a conversational retrieval system built with Flowise/LangChain.

## Workflow Overview

The chatflow is designed as a **Retrieval-Augmented Generation (RAG)** system that allows users to have conversations with PDF documents. It uses **MistralAI** for natural language generation and **Google Gemini** for creating high-quality text embeddings.

```mermaid
graph TD
    TS[Character Text Splitter] -->|Configures Chunking| PDF[PDF File Loader]
    PDF -->|Ingests Documents| VS[In-Memory Vector Store]
    EMB[Google Gemini Embeddings] -->|Generates Vectors| VS
    VS -->|Retrieves Context (Top K=4)| CHAIN[Conversational Retrieval QA Chain]
    MODEL[MistralAI Chat Model] -->|Generates Answers| CHAIN
    MEM[Buffer Memory] -->|Provides Chat History| CHAIN
    
    style CHAIN fill:#f9f,stroke:#333,stroke-width:2px
    style MODEL fill:#bbf,stroke:#333,stroke-width:2px
    style VS fill:#bfb,stroke:#333,stroke-width:2px
```

---

## Component Details

### 1. Document Processing
- **Loader**: `Pdf File` 
  - **Usage**: `One document per page` (`perPage`) — parses the PDF file page-by-page to keep page level metadata and structure intact.
- **Text Splitter**: `Character Text Splitter`
  - **Chunk Size**: 1000 characters
  - **Chunk Overlap**: 200 characters
  - **Purpose**: Breaks down large documents into manageable segments while maintaining context through overlap.

### 2. Semantic Search & Storage
- **Embeddings**: `Google Gemini Embedding`
  - **Model**: `gemini-embedding-001`
  - **Task Type**: `RETRIEVAL_DOCUMENT`
- **Vector Store**: `In-Memory Vector Store`
  - **Top K**: `4` (Default number of relevant text chunks fetched per query)
  - **Function**: Temporarily stores document embeddings for fast semantic lookup during queries.

### 3. Language Model
- **Model**: `MistralAI`
  - **Version**: `mistral-tiny`
  - **Temperature**: 0.9 (Allowing for creative and fluid responses)
  - **Streaming**: Enabled

### 4. Logic & Memory
- **Chain**: `Conversational Retrieval QA Chain`
  - **Role**: Coordinates the retrieval of relevant context from the vector store and passes it to the Mistral model along with the user's question and past history.
  - **Prompts**:
    - **Rephrase Prompt**: Dynamically reformulates follow-up queries into standalone questions using chat history.
    - **Response Prompt**: Sets the agent's persona to **"AI Assistant"**. Instructs the model to answer strictly using the provided context, saying *"Hmm, I'm not sure"* if the answer cannot be found, preventing hallucinations.
- **Memory**: `Buffer Memory`
  - **Memory Key**: `chat_history`
  - **Function**: Maintains the state of the conversation, allowing the model to understand follow-up questions.

---

## Configuration Summary

| Feature | Setting / Value |
| :--- | :--- |
| **Primary LLM** | Mistral Tiny (`mistral-tiny`) |
| **Temperature** | 0.9 (Creativity level) |
| **Embedding Model** | Gemini Embedding 001 (`gemini-embedding-001`) |
| **Storage Type** | In-Memory Vector Store |
| **Retriever Top K** | 4 chunks |
| **Splitting Strategy** | Character-based (1000 size / 200 overlap) |
| **PDF Parsing Mode** | One document per page (`perPage`) |
| **Chain Type** | Conversational Retrieval QA |
| **Memory Buffer Key** | `chat_history` |

