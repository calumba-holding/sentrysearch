# sentrysearch

Semantic search over dashcam footage. Type what you're looking for, get a trimmed clip back.

<!-- ![demo](docs/demo.gif) -->

## How it works

sentrysearch splits your dashcam videos into overlapping chunks, embeds each chunk directly as video using Google's Gemini Embedding model, and stores the vectors in a local ChromaDB database. When you search, your text query is embedded into the same vector space and matched against the stored video embeddings. The top match is automatically trimmed from the original file and saved as a clip.

## Installation

```bash
pip install -e .
```

ffmpeg is required for video chunking and trimming. If you don't have it system-wide, the bundled `imageio-ffmpeg` is used automatically.

Set up your Gemini API key:

```bash
cp .env.example .env
# Edit .env and add your key from https://aistudio.google.com/apikey
```

Or export it directly:

```bash
export GEMINI_API_KEY=your-key-here
```

## Usage

### Index footage

```bash
$ sentrysearch index /path/to/dashcam/footage
Indexing file 1/3: front_2024-01-15_14-30.mp4 [chunk 1/4]
Indexing file 1/3: front_2024-01-15_14-30.mp4 [chunk 2/4]
...
Indexed 12 new chunks from 3 files. Total: 12 chunks from 3 files.
```

Options: `--chunk-duration 30` (seconds per chunk), `--overlap 5` (overlap between chunks).

### Search

```bash
$ sentrysearch search "red truck running a stop sign"
  #1 [0.87] front_2024-01-15_14-30.mp4 @ 02:15-02:45
  #2 [0.74] left_2024-01-15_14-30.mp4 @ 02:10-02:40
  #3 [0.61] front_2024-01-20_09-15.mp4 @ 00:30-01:00

Saved clip: ./match_front_2024-01-15_14-30_02m15s-02m45s.mp4
```

Options: `--results N`, `--output-dir DIR`, `--no-trim` to skip auto-trimming.

### Stats

```bash
$ sentrysearch stats
Total chunks:  47
Source files:  12
```

### Verbose mode

Add `--verbose` to either command for debug info (embedding dimensions, API response times, similarity scores).

## How is this possible?

Gemini Embedding 2 can natively embed video — raw video pixels are projected into the same 768-dimensional vector space as text queries. There's no transcription, no frame captioning, no text middleman. A text query like "red truck at a stop sign" is directly comparable to a 30-second video clip at the vector level. This is what makes sub-second semantic search over hours of footage practical.

## Cost

Indexing 1 hour of footage costs ~$0.25 with Gemini's embedding API. Search queries are negligible (text embedding only).

## Compatibility

This works with any dashcam footage in mp4 format, not just Tesla Sentry Mode. The directory scanner recursively finds all `.mp4` files regardless of folder structure.

## Requirements

- Python 3.10+
- `ffmpeg` on PATH, or use bundled ffmpeg via `imageio-ffmpeg` (installed by default)
- Gemini API key ([get one free](https://aistudio.google.com/apikey))
