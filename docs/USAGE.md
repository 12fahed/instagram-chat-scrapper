# Usage Guide

[Back to README](../README.md)

## Contents

- [Requirements](#requirements)
- [1. Capture the thread](#1-capture-the-thread)
- [2. Check what you captured](#2-check-what-you-captured)
- [3. Build the reading view](#3-build-the-reading-view)
- [4. Download the media (optional, slow)](#4-download-the-media-optional-slow)
- [Output](#output)
- [Commands and flags](#commands-and-flags)
- [Known limitations](#known-limitations)

## Requirements

- Python 3.10+ (uses `X | None` type syntax and the walrus operator)
- [Playwright](https://playwright.dev/python/) with Chromium
- An Instagram account that can open the conversation in a browser
- Disk: raw captures run roughly 60 KB per 20-message page, so a 90,000-message
  thread is a few hundred MB of JSON, plus a similar amount again for the HTML

```bash
python -m venv venv
source venv/bin/activate
pip install playwright
playwright install chromium
```

There are no other dependencies. The analyser uses the standard library only.

## 1. Capture the thread

```bash
python -m src.capture
```

A Chromium window opens on the DM inbox. Then, in order:

1. Log in if needed. **Press Enter in the terminal before you go looking for the
   conversation.** See [Known limitations](#known-limitations) for why this matters.
2. Open the conversation you want. The tool waits for it, prints
   `captured the thread query — API paging is available`, and starts.
3. Leave it running. It prints a line per page and stops on its own when it
   reaches the beginning of the thread.

`Ctrl+C` pauses rather than quits: recording stops, the browser is yours to use,
and `r` resumes from the same cursor. A second `Ctrl+C` stops the run. The browser
is never closed automatically. On a crash, or at the top of the thread, it stays
open and reports what happened.

If the browser dies, run the command again. It offers to continue from the
furthest-back cursor already on disk instead of starting over.

## 2. Check what you captured

```bash
python -m src.analyze report
```

Check four things:

- **UNHANDLED** should say `every message matched a known shape`. If it lists
  message ids, the renderer would show those as "Unsupported". The shape is new
  and the parser needs a branch for it.
- **COVERAGE** shows the date range, and `unique messages` vs `nodes seen`.
- **CONTINUITY → cursor chain** should read `all N paged requests follow a captured page`,
  which means no page was skipped.
- **rendered as** gives a count per message kind. `unknown` should be zero.

## 3. Build the reading view

```bash
python -m src.analyze build --me <your-username>
```

Open `data/out/chat.html`. Images will be missing at this stage. That is expected.

## 4. Download the media (optional, slow)

```bash
python -m src.analyze build --me <your-username> --media
```

This makes one request per image, serially. Expect failures on older messages:
Instagram's CDN links are signed and expire within days, so anything shared more
than a few days ago is usually gone. Failures leave a visible placeholder rather
than a gap. Downloads are cached by URL, so re-running only fetches what is missing.

## Output

| Path | Contents |
| --- | --- |
| `data/captures/` | Every raw response, exactly as received, written as it arrives |
| `data/out/chat.json` | One normalised, de-duplicated message list |
| `data/out/chat*.html` | A self-contained reading view with reply quotes, reactions, media, date dividers, search and dark mode |
| `data/out/report.txt` | What was captured, whether it is complete, and anything not understood |

All of `data/` is gitignored.

## Commands and flags

### `python -m src.capture`

| Flag | Description |
| --- | --- |
| *(none)* | Replay the thread's own GraphQL query, cursor by cursor. Constant memory, no limit on thread length. |
| `--scroll` | Drive the UI by scrolling instead. Slower, and the renderer runs out of memory at roughly 20,000 messages. Kept as a fallback if a replayed request is ever refused. |
| `--images` | Let the page load images, video and fonts. Off by default, since media URLs come from the JSON and loading them only costs memory. |
| `--delay N` | Seconds between paged requests (default `1.2`, jittered). Raise it if Instagram starts refusing requests. |

### `python -m src.analyze report [input]`

Describes the captures and flags anything unrecognised. `input` defaults to
`data/captures`. `-o/--out` sets where `report.txt` is written (default `data/out`).

### `python -m src.analyze build [input]`

| Flag | Description |
| --- | --- |
| `--me <id>` | Which participant is you: a username, numeric id or display name. Decides which side of the conversation appears on the right. |
| `--title <text>` | Page title. |
| `--split auto\|month\|year\|none` | How to divide the archive. `auto` (default) keeps one page under 3,000 messages, splits by year above that, and by month above 20,000. |
| `--media` | Download images and avatars into `data/out/media/` and point the pages at the local copies. |
| `--media-video` | With `--media`, also download video originals. Large, and the least likely to still resolve. |
| `-o/--out <dir>` | Output directory (default `data/out`). |

## Known limitations

**The newest ~20 messages can be missing.** Scrolling the inbox list makes Instagram
prefetch a thread's most recent messages. If that happens before you press Enter,
those messages are already cached in the browser, so opening the conversation does
not re-request them and they never appear in a recorded response. *Press Enter
before you scroll the inbox at all.* If you have already captured, scroll up and
back down inside the open conversation to force a refetch.

**Media expires.** Instagram's CDN URLs are signed and short-lived. The archive
always keeps the URLs, but `--media` can only download what is still being served,
which in practice means recent messages.

**View-once media is unrecoverable.** Consumed view-once photos come back as a
12×12 blurred placeholder with no full-size version anywhere. These are shown as
"Photo unavailable".

**One thread per capture directory.** `data/captures/` is assumed to hold a single
conversation. Archive a second one into a different directory, or move the first
out of the way.

**Instagram can change the query at any time.** This reads an internal, unversioned
API. If the shape changes, `analyze report` will say so under UNHANDLED rather than
failing silently.
