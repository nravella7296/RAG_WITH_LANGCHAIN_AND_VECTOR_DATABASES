# langc-course

Hands-on exercises for learning [LangChain](https://python.langchain.com/) and [LangGraph](https://langchain-ai.github.io/langgraph/), using OpenAI and Anthropic chat models.

## Requirements

- Python 3.13+
- [uv](https://docs.astral.sh/uv/) for dependency management
- An OpenAI API key and an Anthropic API key

## Setup

1. Install dependencies:

   ```bash
   uv sync
   ```

2. Create a `.env` file in the project root with your API keys:

   ```env
   OPENAI_API_KEY=your-openai-key
   ANTHROPIC_API_KEY=your-anthropic-key
   ```

   The scripts load these with `python-dotenv`. Keep `.env` out of version control.

## Project structure

```
.
├── main.py                  # Setup check: prints versions and calls both LLMs
├── document_loaders.py      # Document loader examples
├── text_splitters.py        # Text splitting and chunking strategies
├── raw_emb.py               # Raw OpenAI SDK: chat completion + embeddings
├── embeddings_deep.py       # Embeddings: batching, cosine similarity, caching
├── vector_stores.py         # Chroma vector store and retriever examples
├── rag_pipeline.py          # Retrieval-augmented generation (RAG) pipelines
├── prod_hybrid_search.py    # Hybrid search: BM25 + vector with Reciprocal Rank Fusion
├── docs/
│   └── langchain_demo.pdf   # Sample PDF used by the PDF loader example
├── src/langc_course/        # Package entry point (langc-course script)
├── pyproject.toml
└── uv.lock
```

## Usage

### Verify your setup

```bash
uv run main.py
```

This prints the installed `langchain-core` and `langgraph` versions, then sends a test prompt to `ChatOpenAI` (`gpt-4o-mini`) and `ChatAnthropic` (`claude-sonnet-5-5`). If both respond, your keys and environment work.

### Document loaders

```bash
uv run document_loaders.py
```

`document_loaders.py` shows different ways to load data into LangChain `Document` objects:

| Function | What it shows |
| --- | --- |
| `load_text_file()` | Loading a plain text file with `TextLoader` |
| `web_loader()` | Scraping a web page with `WebBaseLoader` (BeautifulSoup) |
| `lazy_loader()` | Loading a directory of files one at a time with `DirectoryLoader.lazy_load()` |
| `doc_structure()` | Building a `Document` by hand to show `page_content` and `metadata` |
| `pdf_loader(path)` | Loading a PDF page by page with `PyPDFLoader` |

By default only `pdf_loader("./docs/langchain_demo.pdf")` runs. To try the others, uncomment them in the `if __name__ == "__main__":` block.

### Text splitters

```bash
uv run text_splitters.py
```

`text_splitters.py` compares chunking strategies for RAG on a sample Markdown article, a Python code snippet, and the sample PDF. It needs no API keys.

| Function | What it shows |
| --- | --- |
| `recursive_splitter()` | Splitting text with `RecursiveCharacterTextSplitter` and custom separators |
| `chunk_size_comparison()` | How chunk counts change at sizes 200, 500, and 1000 with 20% overlap |
| `overlap_importance()` | Chunk boundaries with and without `chunk_overlap` |
| `markdown_splitter()` | Splitting on headers with `MarkdownHeaderTextSplitter`, keeping them as metadata |
| `code_splitter()` | Language-aware splitting of Python code with `RecursiveCharacterTextSplitter.from_language` |
| `document_splitter()` | Loading `docs/langchain_demo.pdf` with `PyPDFLoader` and splitting it with `split_documents`, preserving page metadata |

By default only `document_splitter()` runs; uncomment the others in the `if __name__ == "__main__":` block.

### Raw OpenAI embeddings

```bash
uv run raw_emb.py
```

`raw_emb.py` calls the OpenAI SDK directly, without LangChain. It builds a `gpt-4o-mini` chat completion (printing is commented out) and creates an embedding with `text-embedding-3-large`, then prints the response and the vector length.

### Embeddings in depth

```bash
uv run embeddings_deep.py
```

`embeddings_deep.py` looks at embeddings directly, using `OpenAIEmbeddings` (`text-embedding-3-large`) and NumPy:

| Function | What it shows |
| --- | --- |
| `basic_embeddings()` | Embedding one query with `embed_query`, then printing its dimensions, first values, and norm |
| `batch_embeddings()` | Embedding several texts at once with `embed_documents` |
| `similarity_search()` | Ranking documents against a query by cosine similarity, computed by hand |
| `embedding_caching()` | Caching embeddings with `CacheBackedEmbeddings` and a `LocalFileStore`, so a repeated call reads from the cache instead of the API |

By default only `embedding_caching()` runs; uncomment the others in the `if __name__ == "__main__":` block.

### Vector stores (Chroma)

```bash
uv run vector_stores.py
```

`vector_stores.py` embeds a small set of sample `Document`s with `OpenAIEmbeddings` (`text-embedding-3-large`) and stores them in Chroma:

| Function | What it shows |
| --- | --- |
| `chroma_basics()` | Creating a Chroma store from documents and running `similarity_search` |
| `similarity_search_with_scores()` | `similarity_search_with_score`, converting distance to a similarity score |
| `metadata_filtering()` | Comparing search results with and without a metadata `filter` |
| `as_retriever()` | Using the store as a retriever, with plain similarity and MMR search |
| `persist_chroma()` | Persisting to `./chroma_db/` and reloading from disk |
| `exercise_vector_store_setup()` | Exercise: split texts with `RecursiveCharacterTextSplitter`, store in Chroma, return a retriever |

Most examples use a temporary directory, so nothing is left on disk. `persist_chroma()` writes to `./chroma_db/`, which is git-ignored. By default only `exercise_vector_store_setup()` runs; uncomment the others in the `if __name__ == "__main__":` block.

### RAG pipelines

```bash
uv run rag_pipeline.py
```

`rag_pipeline.py` builds retrieval-augmented generation chains with LCEL. It splits a small built-in Markdown knowledge base about LangChain with `RecursiveCharacterTextSplitter`, embeds the chunks into Chroma with `OpenAIEmbeddings` (`text-embedding-3-large`), and answers questions with `gpt-4o-mini` (via `init_chat_model`):

| Function | What it shows |
| --- | --- |
| `demo_basic_rag()` | A basic chain: retriever → prompt → LLM → `StrOutputParser`, answering only from the retrieved context |
| `demo_rag_with_sources()` | Labeling retrieved chunks with their `source` metadata so the answer cites them |
| `demo_rag_with_fallback()` | Making the model answer with a fixed fallback message when the context doesn't contain the answer |
| `demo_structured_rag()` | Returning a Pydantic `RAGResponse` (answer, confidence, sources, follow-up) via `with_structured_output` |
| `exercise_document_qa()` | Exercise: a `DocumentQA` class that indexes any text and answers questions with a confidence rating |

By default only `demo_basic_rag()` runs; uncomment the others in the `if __name__ == "__main__":` block.

### Hybrid search (BM25 + vector)

```bash
uv run prod_hybrid_search.py
```

`prod_hybrid_search.py` compares three ways to retrieve from a small set of support documents that mix plain prose with exact identifiers such as `SKU-7742X`, `E_CONN_REFUSED`, and `WCAG`:

- **Vector**: a Chroma retriever over `OpenAIEmbeddings` (`text-embedding-3-large`), good at meaning-based questions.
- **BM25**: `BM25Retriever` keyword matching, good at exact codes and acronyms.
- **Hybrid**: `hybrid_retrieve()` merges both result lists with weighted Reciprocal Rank Fusion (RRF), the same approach `EnsembleRetriever` uses internally.

It runs five test queries and prints the top 3 results from each method side by side. Unlike the other scripts, it runs everything at import time; there's no `__main__` block to edit.

## Dependencies

- `langchain`, `langchain-core`, `langchain-community`: core framework and community loaders
- `langchain-openai`, `langchain-anthropic`, `anthropic`: model integrations
- `langchain-chroma`: Chroma vector store integration
- `langchain-ollama`: Ollama integration for local models
- `langgraph`: graph-based agent workflows
- `pypdf`, `beautifulsoup4`: PDF and HTML parsing for loaders
- `rank-bm25`: BM25 keyword scoring for `BM25Retriever`
- `python-dotenv`: loads API keys from `.env`
