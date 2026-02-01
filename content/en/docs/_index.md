---
title: RsyncUI - a GUI for rsync
linkTitle: Documentation
menu: { main: { weight: 20 } }
---

*RsyncUI* is a macOS application built with Swift and SwiftUI for macOS Sonoma and later. The command-line tool rsync performs the synchronization; RsyncUI provides the GUI on top.

RsyncUI is free and open source. Please review the MIT license before you start using it. The project will remain open and free.

<div class="alert alert-secondary" role="alert">

RsyncUI is *digitally signed* and *notarized* by Apple to protect against tampering. It is not sandboxed—sandboxing would block access to hidden directories such as `.ssh` or `.rsyncosx` and prevent using Homebrew's rsync, making RsyncUI ineffective.

</div>

### Concealing Actions

See *Getting started* for details on Concealing Actions. Toggle action visibility on the main toolbar with `⌘S`.

### Privacy statement

RsyncUI is a desktop-only app with no server component. It does not send logs off your Mac. Task data and logs stay local; you can back them up to your Documents directory from Settings. Data leaves your Mac only when you enable tasks that target a remote server, and you are responsible for securing that data.

### Changelog and Installation

For current updates, see the [changelog](/blog/). RsyncUI ships as a Universal macOS Binary and runs natively on Apple Silicon and Intel Macs.

RsyncUI can be installed via Homebrew or downloaded from [GitHub](https://github.com/rsyncOSX/RsyncUI/releases/).
```bash
brew install --cask rsyncui
```

If installed via Homebrew, the SHA-256 hash is automatically verified. For downloads from GitHub, please verify the SHA-256 hash manually.