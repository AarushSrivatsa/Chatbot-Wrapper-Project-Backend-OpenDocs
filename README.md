# Chatbot Wrapper Project Backend OpenDocs

A production-ready FastAPI backend for multi-user conversational AI applications with advanced retrieval-augmented generation (RAG), web search capabilities, and intelligent document processing.

---

## 🌟 Overview

This API provides a comprehensive backend service for building AI-powered chat applications. It manages users, conversations, and messages while supporting conversation-scoped retrieval-augmented generation with optional reranking and web search integration.

**Key Differentiator:** Unlike typical chatbot backends that focus on either RAG or web search, this system seamlessly integrates both capabilities, allowing conversations to leverage uploaded documents AND real-time web information simultaneously.

**Base API URL:** [https://chatbotwrapperprojectbackend.onrender.com](https://chatbotwrapperprojectbackend.onrender.com)

**Interactive Documentation:** [https://chatbotwrapperprojectbackend.onrender.com/docs](https://chatbotwrapperprojectbackend.onrender.com/docs)

---

## ✨ Core Features

### Authentication & User Management
- **JWT-based authentication** with dual-token system (access tokens + refresh tokens)
- Secure user registration and login
- Access tokens for API authentication
- Refresh tokens for obtaining new access and refresh tokens
- Secure logout with token invalidation
- Password hashing with industry-standard algorithms
- User-scoped data isolation

### Conversation Management
- **Multi-user support** with isolated conversation spaces
- Create, list, and delete conversations
- Conversation-level RAG configuration
- Persistent message history

### Advanced RAG Pipeline
- **Universal document support** - Upload any document type (PDF, DOCX, TXT, etc.)
- **Recursive character text splitting** with custom separators for optimal chunk boundaries
- **Local embeddings** via Ollama (nomic-embed-text:v1.5) - no external API calls
- **Optional ms-marco-MiniLM-L-12-v2 reranker** for enhanced retrieval precision
- Configurable chunk size (400 chars) and overlap (75 chars)
- Similarity-based retrieval with BASE_K=20, refined to TOP_N=5
- Conversation-scoped vector storage in Pinecone

### Quad Web Intelligence (Tavily)
- **Search** - Real-time queries with AI-ranked results
- **Extract** - Clean and get content from given URL
- **Crawl** - Graph-based traversal with natural language instructions
- **Map** - Complete website structure visualization

### Message System
- Text message support
- Document upload and processing
- Message history retrieval

---

## 🛠️ Tech Stack

| Category | Technology | Purpose |
|----------|-----------|---------|
| **Backend Framework** | FastAPI | High-performance async API |
| **Authentication** | JWT | Secure token-based auth |
| **ORM** | SQLAlchemy | Database abstraction layer |
| **Database** | PostgreSQL (Supabase) | Relational data storage |
| **Vector Database** | Pinecone | Semantic search & embeddings |
| **LLM Provider** | Groq | Ultra-fast inference |
| **Embeddings** | Ollama (nomic-embed-text:v1.5) | Local semantic embeddings |
| **Reranker** | ms-marco-MiniLM-L-12-v2 | Optional precision reranking |
| **Text Splitting** | RecursiveCharacterTextSplitter | Intelligent document chunking |
| **Web Search** | Tavily | Real-time web crawling & search |
| **Orchestration** | LangChain | RAG pipeline framework |

---

## 📡 API Routes

### Swagger UI :-
<img width="1707" height="841" alt="image" src="https://github.com/user-attachments/assets/957201cd-4a56-4d31-9452-57ce6887ad88" />


### Users
- `POST /users/register` - Register new user
- `POST /users/login` - Login and get JWT tokens (access + refresh)
- `POST /users/refresh` - Refresh both access and refresh tokens
- `POST /users/logout` - Logout and invalidate tokens

### Conversations (Protected)
- `GET /conversations/` - List all conversations for authenticated user
- `POST /conversations/` - Create new conversation
- `DELETE /conversations/{conversation_id}` - Delete conversation

### Messages (Protected)
- `GET /conversations/{conversation_id}/messages/` - Get all messages in conversation
- `POST /conversations/{conversation_id}/messages/` - Send message and get AI response
- `POST /conversations/{conversation_id}/messages/document` - Upload document to conversation

All protected routes require authentication via Bearer token in the Authorization header.

**For detailed API documentation, request/response schemas, and interactive testing, visit the Swagger UI at:** [https://chatbotwrapperprojectbackend.onrender.com/docs](https://chatbotwrapperprojectbackend.onrender.com/docs)

---

## 🧠 RAG Pipeline Architecture

```
Document Upload
    ↓
Text Extraction
    ↓
Recursive Character Splitting (400 chars, 75 overlap)
    ↓
Embedding Generation (Ollama nomic-embed-text:v1.5)
    ↓
Vector Storage (Pinecone)
    ↓
User Query → Vector Similarity Search
    ↓
Top-K Retrieval (BASE_K=20)
    ↓
Optional Reranking (ms-marco-MiniLM-L-12-v2, TOP_N=5)
    ↓
Context Injection → LLM Generation (Groq)
```

### RAG Components

#### 1. Document Chunking
**RecursiveCharacterTextSplitter** ensures intelligent splitting:
- Preserves semantic boundaries
- Chunk size: **400 characters**
- Overlap: **75 characters**
- Custom separators: `["\n\n", "\n", ".", ",", " ", ""]`
- Recursive splitting maintains document structure

#### 2. Embedding Generation
**Ollama nomic-embed-text:v1.5** provides:
- **Local embedding generation** (no external API calls)
- High-quality semantic embeddings
- Fast inference with GPU acceleration (if available)
- Privacy-preserving (data never leaves your infrastructure)
- Cost-effective for high-volume applications

#### 3. Vector Storage
**Pinecone** features:
- Conversation-scoped namespaces
- Fast similarity search
- Metadata filtering
- Scalable to millions of vectors

#### 4. Retrieval & Reranking
**Two-stage retrieval (optional):**
1. **Broad retrieval:** BASE_K=**20** candidates from vector search
2. **Precision reranking:** Optional reranking with **ms-marco-MiniLM-L-12-v2** to TOP_N=**5** most relevant chunks

**Reranking toggle:**
- Set `USE_RERANKING=True` in configuration to enable the reranking stage
- When disabled, the system returns the top 5 results directly from vector search
- Reranking improves precision but adds latency

**ms-marco-MiniLM-L-12-v2 advantages:**
- Cross-encoder architecture for accurate relevance scoring
- Trained on MS MARCO dataset for passage ranking
- Lightweight model suitable for real-time applications
- Significantly improves precision over vector search alone

#### 5. Context Injection
Retrieved context is injected into the system prompt before sending to Groq LLM for generation.

---

## 🌐 Web Search Integration

### Tavily Features

- **Real-time search:** Query the web for current information
- **Content extraction:** Clean, formatted content from web pages
- **Source tracking:** Maintain citation information
- **Result synthesis:** Combine multiple sources intelligently

### Use Cases

1. **Current Events:** Questions requiring up-to-date information
2. **Fact Checking:** Verify information against web sources
3. **Supplemental Context:** Enhance RAG responses with web data
4. **Research Queries:** Gather information from multiple sources

---

## 🔒 Security

- **Password Hashing:** All passwords hashed using secure algorithms
- **JWT Tokens:** Time-limited token expiration
- **User Isolation:** Strict database-level access controls
- **Input Validation:** Pydantic schemas validate all inputs
- **SQL Injection Protection:** SQLAlchemy ORM prevents injection attacks

---

## 🙏 Acknowledgments

- **FastAPI** for the excellent web framework
- **LangChain** for RAG orchestration tools
- **Groq** for lightning-fast inference
- **Ollama** for local embedding generation
- **Pinecone** for vector database infrastructure
- **Tavily** for web search capabilities
- **ms-marco-MiniLM-L-12-v2** for optional reranking

---

**Built with modern Python tools and best practices.**
