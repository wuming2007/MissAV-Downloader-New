# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Miyuki** — a Python CLI tool for downloading videos from MissAV. Downloads video as JPEG frame sequences via m3u8 playlists, then assembles them into MP4 files. Optionally uses ffmpeg for better quality output.

## Build & Run Commands

```bash
# Install locally for development
pip install -e .

# Install from PyPI
pip install miyuki

# Run CLI
miyuki -h

# Example usage
miyuki -urls https://missav.ai/some-video -ffmpeg
miyuki -plist "https://missav.ai/search/query" -limit 10 -ffmpeg
miyuki -search sw-950 -ffcover
miyuki -file /path/to/urls.txt -ffmpeg
```

There is no test suite — `test/test.py` is a manual integration example, not an automated test.

## Architecture

Single-file application: all logic lives in `miyuki/miyuki-new.py` (~728 lines).

**Note:** The entry point in `setup.py` references `miyuki.miyuki:main`, but the actual file is named `miyuki-new.py`. This mismatch means `pip install -e .` will fail to find the module. The file would need to be renamed to `miyuki.py` or the entry point updated.

### Key Flow

1. **URL acquisition** — from one of 5 mutually exclusive sources: `-urls`, `-auth`, `-plist`, `-search`, `-file`
2. **UUID extraction** — regex parses video page HTML for UUID embedded in obfuscated JS
3. **M3U8 resolution** — fetches playlist, selects quality (auto or user-specified via `-quality`)
4. **Multi-threaded download** — downloads JPEG segments in parallel using `os.cpu_count()` threads
5. **Assembly** — concatenates JPEGs into MP4 (raw concatenation or ffmpeg)
6. **Tracking** — records completed URLs in `downloaded_urls_miyuki.txt` to avoid re-downloads

### Key Dependencies

- **curl_cffi** — HTTP client with CloudFlare bypass (the only pip dependency)
- **ffmpeg** — optional external binary for quality optimization and cover embedding
- **Python >= 3.9**

### Important Patterns

- `https_request_with_retry()` — central HTTP function with configurable retry/delay/timeout
- UUID regex pattern: `r'm3u8\|([a-f0-9\|]+)\|com\|surrit\|https\|video'`
- Video segments URL format: `https://surrit.com/{UUID}/{resolution}/video{N}.jpeg`
- `ThreadSafeCounter` — progress tracking across download threads
- Domain currently used: `missav.ws` (for API login) / `missav.ai` (for content)

### Output Files

- `movies_folder_miyuki/` — download output directory
- `downloaded_urls_miyuki.txt` — download history
- `error_records_miyuki.txt` — failure log with timestamps

## Publishing

Automated via GitHub Actions (`.github/workflows/python-publish.yml`) — triggers on release, publishes to PyPI.
