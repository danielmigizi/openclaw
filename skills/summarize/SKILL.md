---
name: summarize
description: Summarize or extract text/transcripts from URLs, podcasts, and local files (great fallback for “transcribe this YouTube/video”).
homepage: https://summarize.sh
metadata:
  {
    "openclaw":
      {
        "emoji": "🧾",
        "requires": { "bins": ["summarize"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "steipete/tap/summarize",
              "bins": ["summarize"],
              "label": "Install summarize (brew)",
            },
          ],
      },
  }
---

# Summarize

Fast CLI to summarize URLs, local files, and YouTube links.

## When to use (trigger phrases)

Use this skill immediately when the user asks any of:

- “use summarize.sh”
- “what’s this link/video about?”
- “summarize this URL/article”
- “transcribe this YouTube/video” (best-effort transcript extraction; no `yt-dlp` needed)

## Quick start

```bash
summarize "https://example.com" --model auto
summarize "/path/to/file.pdf" --model auto
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto
```

## YouTube: summary vs transcript

Best-effort transcript (URLs only):

```bash
summarize "https://youtu.be/dQw4w9WgXcQ" --youtube auto --extract
```

If the user asked for a transcript but it’s huge, return a tight summary first, then ask which section/time range to expand.

## Model + keys

Set the API key for your chosen provider:

- OpenAI: `OPENAI_API_KEY`
- Anthropic: `ANTHROPIC_API_KEY`
- xAI: `XAI_API_KEY`
- Google: `GEMINI_API_KEY` (aliases: `GOOGLE_GENERATIVE_AI_API_KEY`, `GOOGLE_API_KEY`)
- OpenRouter: `OPENROUTER_API_KEY`
- NVIDIA: `NVIDIA_API_KEY` (nvidia/... model aliases)

Default model is `auto` if none is set.

## X/Twitter

Summarize uses `xurl` (preferred) to fetch tweet/thread content, with `bird` as fallback.

## Transcription (audio/video)

- AssemblyAI (remote): set `ASSEMBLYAI_API_KEY` for accurate cloud transcription.
- Gemini Files API: used automatically for audio/video files when `GEMINI_API_KEY` is set.
- Large files (>30 MB audio): handled via Groq Whisper chunking when `GROQ_API_KEY` is set.

## Useful flags

- `--length short|medium|long|xl|xxl|<chars>`
- `--max-output-tokens <count>`
- `--timeout <seconds>` (per-request timeout)
- `--width <cols>` (output column width)
- `--extract` (URLs only)
- `--json` (machine readable)
- `--firecrawl auto|off|always` (fallback extraction)
- `--youtube auto|web|no-auto|apify|yt-dlp`

## Config

Optional config file: `~/.summarize/config.json`

```json
{ "model": "openai/gpt-5.2" }
```

Optional services:

- `FIRECRAWL_API_KEY` for blocked sites
- `APIFY_API_TOKEN` for YouTube fallback
