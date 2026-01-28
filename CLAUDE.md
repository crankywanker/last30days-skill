# CLAUDE.md

## Project Overview

A Claude Code skill that researches topics across Reddit, X (Twitter), and the web from the last 30 days, synthesizing findings into actionable insights. Uses real engagement metrics (upvotes, likes, comments) to surface what people actually care about.

## Quick Commands

```bash
# Run with mock data (no API keys needed)
python3 scripts/last30days.py "topic" --mock --emit=compact

# Run tests
python3 -m pytest tests/ -v

# Direct execution with options
python3 scripts/last30days.py "topic" --emit=compact --sources=auto
# Flags: --quick, --deep, --mock, --debug, --include-web
# Emit modes: compact, json, md, context, path
```

## Architecture

**Pipeline flow**: Discovery → Enrichment → Normalization → Scoring → Deduplication → Rendering

```
scripts/
├── last30days.py          # CLI entry point, orchestrates pipeline
└── lib/
    ├── openai_reddit.py   # OpenAI Responses API + web_search for Reddit
    ├── xai_x.py           # xAI Responses API + x_search for X/Twitter
    ├── websearch.py       # Claude WebSearch (fallback, no key needed)
    ├── reddit_enrich.py   # Fetches real Reddit JSON for engagement metrics
    ├── normalize.py       # Raw API → canonical schema with date filtering
    ├── score.py           # Engagement-aware scoring (relevance + recency + engagement)
    ├── dedupe.py          # Jaccard similarity on 3-grams, threshold 0.7
    ├── render.py          # Output formatting (compact/json/md/context)
    ├── schema.py          # Dataclasses: RedditItem, XItem, WebSearchItem, Report
    ├── cache.py           # 24-hour TTL cache at ~/.cache/last30days/
    ├── env.py             # Loads API keys from ~/.config/last30days/.env
    ├── http.py            # Stdlib HTTP client with retry logic
    ├── models.py          # Auto-selects latest OpenAI/xAI models
    ├── dates.py           # Date range utilities and confidence scoring
    └── ui.py              # Terminal progress display
```

## Key Design Decisions

- **Stdlib only** - No external dependencies (urllib, json, dataclasses)
- **Parallel searches** - Reddit & X run concurrently via ThreadPoolExecutor
- **Graceful degradation** - Missing API keys → WebSearch fallback
- **Engagement-aware scoring** - Combines relevance (45%), recency (25%), engagement (30%)
- **Hard date filtering** - Excludes items outside 30-day range server-side

## Configuration

API keys in `~/.config/last30days/.env`:
```
OPENAI_API_KEY=sk-...      # For Reddit research
XAI_API_KEY=xai-...        # For X/Twitter research
```

Debug mode: `LAST30DAYS_DEBUG=1`

## Testing

Tests use mock fixtures from `fixtures/` directory. All tests run without API keys:
- `test_score.py` - Engagement scoring and normalization
- `test_normalize.py` - Raw → canonical schema conversion
- `test_dedupe.py` - Duplicate detection via Jaccard similarity
- `test_dates.py` - Date range and confidence scoring
- `test_cache.py` - Cache TTL and serialization

## Code Patterns

- Type hints on all functions
- Docstrings with Args/Returns format
- Dataclasses with `.to_dict()` for serialization
- Error containment (enrichment failures don't crash pipeline)
- Test naming: `test_<function>_<scenario>`

## Output Locations

- `~/.local/share/last30days/out/` - Research outputs (report.md, report.json)
- `~/.cache/last30days/` - Query cache (24h TTL)
