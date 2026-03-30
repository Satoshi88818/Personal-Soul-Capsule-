
# SoulCapsule v6

**A Personal Multimodal Memory Vault. Simple. Beautiful. Alive.**

SoulCapsule is a lightweight, fully local, privacy-first tool for capturing, organizing, and meaningfully retrieving your life memories - using text, photos, and voice notes. It combines semantic vector search (via embeddings), mood tracking, automatic tagging, media management, and versioning into one elegant package.

No cloud. No subscriptions. Runs entirely on your machine.

![SoulCapsule Concept](https://picsum.photos/id/1015/800/400)  
*(Visualizing a serene personal memory archive)*

## ✨ What's New in v6

- **Media Garbage Collection** — `soulcapsule gc` command purges orphaned media files. Hard delete now supports `--purge-media`.
- **Soft Negative Prompt Scoring** — Negative queries apply a continuous penalty instead of hard filtering for smoother exclusions.
- **Improved Duplicate Detection** — Checks top-3 nearest candidates for better accuracy.
- **Configurable Mood Weighting** — Switch between symmetric intensity boost (`abs`) or signed valence (`signed`) via `SC_MOOD_WEIGHT_SIGNED`.
- **Recency Boost** — Optional exponential decay reward for recent memories (`SC_RECENCY_HALFLIFE` or `--recency` flag).
- **Static HTML Export** — `soulcapsule export --format html` creates a beautiful, self-contained, mobile-friendly archive (no server needed).
- **Hierarchical Tags** — Support for `category:value` syntax (e.g., `place:paris`, `with:alice`).
- All v5 features retained with full backward compatibility.

## Features

- **Multimodal Capture**
  - Rich text memories with mood and intensity.
  - Photos (CLIP embeddings with PCA projection).
  - Audio notes (Whisper transcription + embedding fusion).
  - Automatic entity-based tagging via spaCy.

- **Intelligent Search**
  - Semantic (vector) search with resonance scoring.
  - Multimodal queries (text + image + audio).
  - Mood filtering, date ranges, minimum resonance.
  - Soft negative prompts and optional recency boost.

- **Memory Management**
  - Versioning with `parent_id` linkage (updates create new versions).
  - Soft-delete + restore.
  - Hard delete with optional media purge.
  - "Surprise Me" — surfaces forgotten (older) memories.

- **Visualization & Insights**
  - Mood drift timeline (emotional arc).
  - Vault statistics and health checks (`doctor`).

- **Export & Backup**
  - JSON, ZIP (with media), or self-contained HTML.
  - Automatic timestamped backups.

- **CLI + Web UI**
  - Rich Typer CLI with beautiful tables.
  - Clean Streamlit interface.

## Installation

```bash
pip install sentence-transformers faiss-cpu openai-whisper Pillow torch torchaudio pydub numpy \
            rich streamlit typer python-dotenv spacy plotly pandas scikit-learn \
            open-clip-torch jinja2

# Download spaCy model
python -m spacy download en_core_web_sm
```

Clone or download the project files (`config.py`, `core.py`, `cli.py`, `ui.py`, templates, `.env` example).

## Quick Start

1. **Initialize** (automatically done on first use):
   ```bash
   python cli.py doctor
   ```

2. **Add a memory**:
   ```bash
   python cli.py add "The first time I saw the ocean at sunrise" \
     --mood joy --intensity 1.8 --tags "place:sydney,with:family"
   ```

3. **Search**:
   ```bash
   python cli.py search "beach memories" --mood joy --recency
   ```

4. **Launch the beautiful web UI**:
   ```bash
   streamlit run ui.py
   ```

5. **Export a portable archive**:
   ```bash
   python cli.py export --format html
   ```

## Project Structure

```
soulcapsule/
├── config.py                 # Constants, mood maps, env config
├── core.py                   # All business logic (DB, FAISS, embeddings, GC)
├── cli.py                    # Typer CLI
├── ui.py                     # Streamlit web UI
├── templates/export.html.j2  # HTML export template
├── .env                      # Optional overrides
├── soulcapsule.db            # SQLite database
├── soulcapsule.faiss         # FAISS index
├── soulcapsule_media/        # Content-addressed photos, audio, thumbnails
└── soulcapsule_backups/      # Auto backups
```

## Configuration (.env)

Key options (full list in `.env` example):

```env
SC_MOOD_WEIGHT_SIGNED=abs          # or "signed"
SC_RECENCY_HALFLIFE=0              # days (0 = disabled)
SC_NEG_PENALTY=0.5
SC_DUP_THRESHOLD=0.88
SC_TEXT_MODEL=all-MiniLM-L6-v2
```

## CLI Reference (Selected Commands)

- `add` — Capture memory (supports `--image`, `--audio`, `--mood`, `--tags`)
- `search` — Semantic search (`--mood`, `--image`, `--exclude`, `--recency`)
- `update` — Create new version of a memory
- `delete` / `restore`
- `gc` — Garbage collect orphaned media (`--dry-run`)
- `export --format html|json|zip`
- `surprise` — Surface a forgotten memory
- `stats` / `doctor` / `rebuild` / `fit-pca`

Run `python cli.py --help` or `python cli.py <command> --help` for details.

## Current Limitations

- **Hardware Requirements**: Embedding models (especially CLIP + Whisper) benefit from a GPU. CPU fallback works but is slower for large vaults or image/audio processing.
- **Scale**: Designed for personal use (hundreds to low thousands of memories). Very large vaults may need index optimization (e.g., switching from flat L2 to HNSW).
- **No Multi-User Support**: Single-user local vault only.
- **Limited Query Language**: Search is primarily semantic/vector-based. Advanced Boolean/tag-specific queries (`place:paris AND mood:joy`) are not natively supported yet (though hierarchical tags are stored).
- **Export Portability**: HTML export relies on relative paths to thumbnails/media. For true offline portability across machines, media must travel with the HTML file (or use the ZIP export).
- **No Built-in Encryption**: Sensitive memories are stored in plaintext (DB + media files). Use OS-level encryption or full-disk encryption for privacy.
- **Model Dependencies**: Requires downloading several large ML models on first use. Offline after initial setup.
- **Audio Handling**: Transcription quality depends on Whisper "base" model (good but not state-of-the-art).

## Known Bugs & Issues

- **Path Resolution in GC**: `collect_garbage()` uses `.resolve()` while DB paths may be stored relatively. Moving the vault directory can cause missed orphan detection.
- **HTML Export Media Links**: Thumbnails in the generated HTML assume the output file is opened from the same relative location as the vault's `soulcapsule_media/` folder. Copying just the `.html` file elsewhere breaks images.
- **PCA Bootstrapping**: Uses a random projection until `fit-pca` is run (after ~50+ images). Early image searches may have slightly lower quality.
- **Duplicate Detection Edge Cases**: Still possible to have near-duplicates slip through if similarity falls just below threshold or during concurrent operations.
- **Streamlit Temporary Files**: Uploaded files for add/search are written to temp locations; cleanup is basic and may leave small artifacts on crashes.
- **Index Desync Recovery**: Auto-rebuild triggers on mismatch, but very large vaults can take time during recovery.
- **No Incremental Index Updates**: Full rebuild required in some desync scenarios.

These are documented in the code comments and `doctor()` output where relevant.

## Future Enhancements (v7 Ideas)

- **Hybrid Search**: Combine vector similarity with SQLite Full-Text Search (FTS5) for better keyword precision.
- **Advanced Tag Querying**: Native support for `place:paris` filters and Boolean tag logic.
- **Memory Graph / Linking**: Extend `parent_id` to arbitrary "related" links with a simple visualization.
- **Version Diffs**: Show textual diffs in UI/CLI when viewing memory history.
- **Incremental Indexing & Better ANN**: Support for HNSW or IVF indexes for faster search on large vaults.
- **Optional End-to-End Encryption**: For sensitive entries (with trade-offs on searchability).
- **Mobile PWA**: Enhance static HTML export with offline service worker and better mobile UX.
- **LLM Integration (Optional)**: Local summarization, auto-titling, or gentle reflection prompts (using Ollama or similar, kept optional).
- **Import from Other Tools**: Obsidian, Day One, or photo library importers.
- **Performance Dashboard**: More detailed analytics (e.g., search latency, embedding stats).

## Philosophy

SoulCapsule exists to make your memories **simple to capture**, **beautiful to revisit**, and **alive** through intelligent retrieval. It prioritizes longevity, privacy, and emotional nuance (mood valence, recency, soft exclusions) over flashy features.

It fights digital entropy with garbage collection, versioning, and easy exports — so your personal archive remains clean and usable for decades.

## Contributing & Support

This is a personal project released for the community. Feel free to fork, improve, or submit issues/PRs.

**License**: MIT (assumed — add your preferred license).

**Author Note**: Built with love for preserving the small, meaningful moments of life.

---

*SoulCapsule v6 — March 2026*

Made for those who want their memories to feel like home.
```



