# Small RAG

A fully local Retrieval-Augmented Generation (RAG) app that answers questions about a PDF. No API keys and no paid services: everything runs on your own machine.

## How it works

```
PDF ──► Load ──► Split into chunks ──► Embed ──► LanceDB (vector store)
                                                        │
Question ──► Embed ──► Retrieve top-k chunks ◄──────────┘
                              │
                              ▼
                 Prompt (context + question) ──► Llama 3.2 (Ollama) ──► Answer
```

1. **Load:** `PyPDFLoader` reads the PDF page by page.
2. **Chunk:** `RecursiveCharacterTextSplitter` splits the text into 1,000-character chunks with a 200-character overlap.
3. **Embed:** each chunk becomes a vector using the local HuggingFace model `all-MiniLM-L6-v2`.
4. **Store:** the vectors are saved in a local LanceDB database.
5. **Retrieve:** a question is embedded the same way, and the most similar chunks are pulled back.
6. **Generate:** the retrieved chunks and the question go to Llama 3.2 running in Ollama, which answers using only that context.

## Tech stack

| Component | Tool |
|---|---|
| Orchestration | LangChain (LCEL) |
| PDF loading | `pypdf` via `PyPDFLoader` |
| Embeddings | `sentence-transformers/all-MiniLM-L6-v2` (HuggingFace, runs locally) |
| Vector store | LanceDB |
| LLM | Llama 3.2 via Ollama (runs locally) |
| Environment | `uv` + Jupyter notebook in VS Code |

## Setup

### Prerequisites
- Python 3.10+
- [uv](https://docs.astral.sh/uv/)
- [Ollama](https://ollama.com)

### 1. Clone the repo
```bash
git clone https://github.com/Sharmistaroy/Small-RAG.git
cd Small-RAG
```

### 2. Install dependencies
```bash
uv sync
```
This creates a `.venv` with the exact package versions from `uv.lock`.

### 3. Download the local LLM
Open the Ollama app, then run:
```bash
ollama pull llama3.2
```

### 4. Add a PDF
PDFs aren't included in the repo. Put your own PDF in a `Data/` folder and update the path in the loader cell of the notebook.

### 5. Run the notebook
1. Open the notebook in VS Code.
2. Click **Select Kernel** and choose the `.venv` Python environment.
3. Run all cells. The first run downloads the embedding model (about 90MB).

Keep the Ollama app running while you use the notebook.

## Usage

Change the question in the last cell and rerun it:

```python
print(chain.invoke("Why is building an ML system an iterative process?"))
```

If the answer isn't in the PDF, the model is told to say it doesn't know rather than make something up.

## Configuration

| Setting | Default | Where to change it |
|---|---|---|
| Chunk size / overlap | 1000 / 200 | `RecursiveCharacterTextSplitter(...)` |
| Chunks retrieved per question | 3 | `as_retriever(search_kwargs={"k": 3})` |
| Embedding model | `all-MiniLM-L6-v2` | `HuggingFaceEmbeddings(model_name=...)` |
| LLM | `llama3.2` | `ChatOllama(model=...)`; any model from `ollama list` works |

## Troubleshooting

| Problem | Fix |
|---|---|
| `Connection refused` when asking a question | The Ollama app isn't running. Open it and rerun the cell. |
| `ModuleNotFoundError` | The notebook is on the wrong kernel. Select the `.venv` kernel and restart. |
| `No pyproject.toml found` | Run `uv` commands from inside the project folder. |
| `HF_TOKEN` or `TqdmWarning` messages | Harmless warnings that can be ignored. |

## Possible next steps
- Show the source page number next to each answer
- Load an existing LanceDB table instead of re-embedding on every run
- Add a simple chat UI with Streamlit or Gradio
- Evaluate retrieval quality with a small set of test questions
