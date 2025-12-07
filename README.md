# pdf-library

Local PDF knowledge base with vector search. Extract, embed, and semantically search your PDFs.

## Features

- **Local-first** - Everything runs on your machine, no API costs
- **Vector search** - Semantic search via Ollama embeddings (mxbai-embed-large)
- **Hybrid search** - Combine vector similarity with full-text search
- **iCloud sync** - Default storage in `~/Documents/.pdf-library/`
- **Fast** - SQLite + sqlite-vec for instant queries

## Requirements

- macOS (Apple Silicon recommended)
- [Bun](https://bun.sh) runtime
- [Ollama](https://ollama.ai) for embeddings
- [uv](https://github.com/astral-sh/uv) for PDF extraction

## Setup

```bash
# Clone and setup
git clone https://github.com/joelhooks/pdf-library.git
cd pdf-library
./scripts/setup.sh
```

This will:

1. Install Ollama if needed
2. Pull the `mxbai-embed-large` embedding model
3. Install dependencies
4. Create the library directory

## Usage

### CLI

```bash
# Add a PDF from local path
bun run dev add /path/to/document.pdf

# Add from URL
bun run dev add https://example.com/paper.pdf

# Add with tags
bun run dev add /path/to/document.pdf --tags "ai,agents"

# Add from URL with custom title
bun run dev add https://example.com/paper.pdf --title "Research Paper" --tags "research"

# Search semantically
bun run dev search "context engineering patterns"

# Full-text search
bun run dev search "context engineering" --fts

# List all documents
bun run dev list

# List by tag
bun run dev list --tag ai

# Get document details
bun run dev get "document-title"

# Remove a document
bun run dev remove "document-title"

# Update tags
bun run dev tag "document-title" "new,tags,here"

# Show stats
bun run dev stats

# Check Ollama status
bun run dev check
```

### As a library

```typescript
import { PDFLibrary } from "pdf-library";

const library = new PDFLibrary();

// Add a PDF
await library.add("/path/to/document.pdf", {
  tags: ["ai", "agents"],
});

// Semantic search
const results = await library.search("context engineering patterns");

// Hybrid search (vector + FTS)
const results = await library.search("context engineering", { hybrid: true });

// List documents
const docs = library.list();
```

### OpenCode Tool

Copy `opencode-tool.ts` to `~/.config/opencode/tool/pdf-library.ts` to use as an OpenCode custom tool.

## Configuration

Environment variables:

| Variable           | Default                    | Description              |
| ------------------ | -------------------------- | ------------------------ |
| `PDF_LIBRARY_PATH` | `~/Documents/.pdf-library` | Library storage location |
| `OLLAMA_HOST`      | `http://localhost:11434`   | Ollama API endpoint      |
| `OLLAMA_MODEL`     | `mxbai-embed-large`        | Embedding model          |

## How it works

1. **Extract** - PDF text extracted via `pypdf` (run through `uv`)
2. **Chunk** - Text split into ~512 token chunks with overlap
3. **Embed** - Each chunk embedded via Ollama (mxbai-embed-large, 1024 dims)
4. **Store** - SQLite database with sqlite-vec for vector search + FTS5 for full-text
5. **Search** - Query embedded, compared against chunks via cosine similarity

## Storage

```
~/Documents/.pdf-library/
├── library.db          # SQLite database (vectors, FTS, metadata)
├── downloads/          # PDFs downloaded from URLs
├── extracted/          # Markdown versions of PDFs (optional)
└── originals/          # Copy of original PDFs (optional)
```

## License

MIT
