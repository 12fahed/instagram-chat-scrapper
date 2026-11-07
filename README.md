<div align="center">

# Instagram Chat Scraper

**Archive a complete Instagram direct-message thread as a readable, offline record.**

![Python](https://img.shields.io/badge/python-3.10%2B-3776AB?logo=python&logoColor=white)
![Playwright](https://img.shields.io/badge/playwright-chromium-2EAD33?logo=playwright&logoColor=white)
![Status](https://img.shields.io/badge/status-early-orange)
![License](https://img.shields.io/badge/license-MIT-blue)

</div>

Instagram's official data export reduces a conversation to disconnected rows: replies lose
their context, and shared posts become bare links. This tool records the responses that
Instagram's own web app receives and rebuilds the thread exactly as it appears on screen.
All data remains on your machine.

| Instagram's official export | This archive |
| :---: | :---: |
| ![Official export](docs/images/ig-exported.png) | ![Rebuilt archive](docs/images/scraper-formatter.png) |
| Replies detached, shares reduced to links | Replies quoted inline, shares shown as cards |

## Features

- **Complete threads.** Pages through the full history, including threads of 90,000+ messages.
- **Faithful replies.** Every reply shows the message it answers, with a link back to it.
- **Rich content.** Shared posts, reels, stories, reactions and voice notes are preserved.
- **Self-contained output.** A searchable HTML reading view with dark mode, plus clean JSON.
- **Private by design.** No telemetry, no third-party services, and auth tokens are never saved.

## Quick Start

```bash
pip install playwright && playwright install chromium

python -m src.capture                          # 1. record the thread
python -m src.analyze report                   # 2. verify the capture
python -m src.analyze build --me <username>    # 3. build the archive
```

Open `data/out/chat.html` to read the result. Add `--media` to the build command to download images.

> [!IMPORTANT]
> Press **Enter** in the terminal *before* browsing the inbox, then open the conversation.
> Otherwise the newest messages may be missed. See the [Usage Guide](docs/USAGE.md) for details.

## Documentation

| Document | Contents |
| --- | --- |
| [Usage Guide](docs/USAGE.md) | Step-by-step usage, all commands and flags, known limitations |
| [Overview](docs/OVERVIEW.md) | Background, architecture, testing, privacy and repository layout |
| [Developer Guide](docs/DEVELOPERS.md) | Internals: the query shape, message types, pagination and resuming |
| [Contributing](CONTRIBUTING.md) | How to report bugs and submit fixes |

## Disclaimer

This project is in an early stage and relies on an internal, unversioned Instagram API that
may change without notice. Use it only to archive conversations you are part of, on an account
you own. The other participants have not consented to being archived, so keep all captured
data private.

## License

Released under the [MIT License](LICENSE).
