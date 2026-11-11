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

   Both scripts load these with `python-dotenv`. Keep `.env` out of version control.

## Project structure

```
.
├── main.py                  # Setup check: prints versions and calls both LLMs
├── document_loaders.py      # Document loader examples
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

## Dependencies

- `langchain`, `langchain-core`, `langchain-community`: core framework and community loaders
- `langchain-openai`, `langchain-anthropic`, `anthropic`: model integrations
- `langgraph`: graph-based agent workflows
- `pypdf`, `beautifulsoup4`: PDF and HTML parsing for loaders
- `python-dotenv`: loads API keys from `.env`
