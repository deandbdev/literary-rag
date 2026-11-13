# Literary RAG — style-aware story generation with hybrid retrieval

A local retrieval-augmented generation system that reads an author's stories, extracts a measurable style profile, and writes new stories in that style. Generation is followed by an automatic critique pass that compares the draft against the profile.

> Status: working personal project. No author texts are included in this repository; you bring your own `.txt` files.

<!-- TODO: add a screenshot of the generator page and one of the style profile page -->

## How it works

1. **Ingest** — each story is split into chunks, embedded with BGE-m3, and stored in SQLite (FTS5 for keyword search, vec0 for vectors).
2. **Analyse style** — an LLM produces a JSON style profile for every story; the profiles are then merged into one `style_master` profile for the author.
3. **Retrieve** — a FastAPI service runs hybrid search: BM25 keyword ranking plus vector similarity, with extra weight for story openings and endings. A cross-encoder reranker (BGE-reranker-v2-m3) orders the results.
4. **Generate (layer 1)** — the PHP backend builds a prompt from the style profile and retrieved passages and streams the story to the browser over server-sent events.
5. **Critique (layer 2)** — a second LLM pass compares the draft against the style profile and flags where it drifts.

Each author is a separate profile with its own database, so styles never mix.

## Stack

- **Python:** ingestion, style analysis, FastAPI query server
- **Retrieval:** SQLite FTS5 (BM25) + vec0 vectors, BGE-m3 embeddings, BGE-reranker-v2-m3 (ONNX)
- **Backend:** PHP REST router with SSE streaming
- **Frontend:** plain HTML pages: generator, upload, style profile viewer
- **LLM:** any OpenAI-compatible endpoint (configured in `config.php`)

## Quick start

```powershell
pip install -r python/requirements.txt
copy .env.example .env             # then put your key in .env (any OpenAI-compatible endpoint)
cd python
python init_db.py                  # create the database
# put your .txt files in texts/ (or use upload.html)
cd ..
start.bat                          # starts the Python server on 127.0.0.1:5002
php -S 127.0.0.1:8080              # then open http://127.0.0.1:8080/index.html
```

Set `AUTHOR_NAME` in `config.php` before generating. The ONNX model files (BGE-m3 and the reranker) are large and are not stored in the repository; point the `EXTERNAL_MODELS_DIR` environment variable at a folder containing `bge-m3/` and `bge-reranker/` (default: `models/`). <!-- Dean: add download steps for the two models. -->

## Pipeline files

| File | Role |
|---|---|
| `ingest_texts.py` | chunk, embed, store |
| `analyze_style.py` | per-story style profile, then master profile |
| `query_server.py` | FastAPI hybrid search service |
| `backend/index.php` | REST router, SSE generation, critique pass |
| `index.html`, `upload.html`, `style_profile.html` | the three pages of the UI |

## Notes on data

The system is designed for texts you have the right to use (your own writing, public-domain works, or material you are licensed to process). It stores no copyrighted corpora in this repository.

## Author

Dean Bouzekow — [LinkedIn](https://www.linkedin.com/in/dean-b-513b24162/)
