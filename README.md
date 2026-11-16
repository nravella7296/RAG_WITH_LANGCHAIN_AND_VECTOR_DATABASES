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
├── raw_emb.py               # Raw OpenAI SDK: chat completion + embeddings
├── vector_stores.py         # Chroma vector store and retriever examples
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

### Raw OpenAI embeddings

```bash
uv run raw_emb.py
```

`raw_emb.py` calls the OpenAI SDK directly, without LangChain. It builds a `gpt-4o-mini` chat completion (printing is commented out) and creates an embedding with `text-embedding-3-large`, then prints the response and the vector length.

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

## Dependencies

- `langchain`, `langchain-core`, `langchain-community`: core framework and community loaders
- `langchain-openai`, `langchain-anthropic`, `anthropic`: model integrations
- `langchain-chroma`: Chroma vector store integration
- `langgraph`: graph-based agent workflows
- `pypdf`, `beautifulsoup4`: PDF and HTML parsing for loaders
- `python-dotenv`: loads API keys from `.env`
