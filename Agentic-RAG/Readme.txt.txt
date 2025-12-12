# Python RAG Tutor with Tooling, Caching, and Memory

This project implements a **Retrieval-Augmented Generation (RAG)**-style agent that helps users learn Python concepts and debug small code snippets. It combines:

- An OpenAI chat model (`gpt-3.5-turbo`)
- A retrieval tool over curated Python tutorials
- A Python code execution tool
- Caching to avoid repeated LLM calls
- Conversation memory across runs

The end result is an **interactive Python tutor** that can both *look up documentation* and *run real code* to explain outputs and errors.

---

## What the Agent Does

At a high level, the agent can:

1. **Explain Python concepts**  
   - e.g., “Explain what a dictionary is in Python”
   - e.g., “Compare dictionaries and tuples”

2. **Use retrieval over real docs**  
   - It fetches and indexes content from:
     - `https://docs.python.org/3/tutorial/index.html`
     - `https://realpython.com/python-basics/`
     - `https://www.learnpython.org/`
   - Uses semantic search (FAISS + embeddings) to find the most relevant snippets.

3. **Execute Python code via a tool**  
   - You can paste a snippet (possibly with a bug), and the agent will:
     - Run it in an isolated environment
     - Capture `stdout`
     - Capture errors and traceback
     - Explain what went wrong in natural language

4. **Remember context within a session**  
   - The agent uses LangGraph’s `MemorySaver` and a `thread_id` to carry context across multiple questions.

5. **Cache LLM responses**  
   - Responses are cached to a local SQLite database (`llm_cache.db`), which:
     - Reduces latency on repeated queries
     - Saves on token/compute cost
     - Survives kernel restarts

---

## Tech Stack

- **Language:** Python
- **Model & API:**
  - OpenAI Chat Completions (`gpt-3.5-turbo`)
- **Frameworks:**
  - [LangChain](https://python.langchain.com/)
  - [LangGraph](https://langchain-ai.github.io/langgraph/)
- **Vector Store:**
  - [FAISS](https://github.com/facebookresearch/faiss) (in-memory index)
- **Caching:**
  - `SQLiteCache` from `langchain_community.cache`
- **Retrieval:**
  - `UnstructuredURLLoader` + `RecursiveCharacterTextSplitter`
- **Tools:**
  - `retrieve_context` (RAG retrieval over Python docs)
  - `run_python` (safe-ish execution of small snippets with captured output)

---

## Python Version and Compatibility

> ⚠️ **Important:** This stack currently relies on older Pydantic-v1 semantics in some dependencies.  
> Newer Python versions (like **3.13+ or 3.14+**) can cause compatibility issues, especially with libraries that still expect Pydantic-v1 APIs.

**Recommended setup:**

- **Python 3.10 or 3.11**  
- Avoid bleeding-edge Python versions unless all dependencies are confirmed compatible.

If you see errors like:

- `PydanticImportError: BaseSettings has been moved to the pydantic-settings package`

then your environment is likely on **Pydantic v2 + a recent Python version**, while some libraries expect v1. Downgrading to Python 3.10/3.11 and reinstalling dependencies usually resolves this.

---

## Dependencies

Here is a minimal list of packages you will need (names may change slightly depending on your environment):

- `openai`
- `langchain`
- `langchain-openai`
- `langchain-community`
- `langchain-text-splitters`
- `langgraph`
- `faiss-cpu` (or platform variant)
- `unstructured` (and related HTML parsers, often `lxml`, `beautifulsoup4`, etc.)
- `tqdm`
- `pydantic<2` (or otherwise a version compatible with your LangChain/LangGraph stack, if they require v1)
- `python-dotenv` (optional, if you prefer loading `OPENAI_API_KEY` from a `.env` file)

Example installation (you can adapt as needed):

```bash
pip install \
  openai \
  langchain \
  langchain-openai \
  langchain-community \
  langchain-text-splitters \
  langgraph \
  faiss-cpu \
  unstructured \
  tqdm
