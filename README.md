# axon — Releases

Pre-built binary distributions for [axon](https://github.com/HideakiSolutions/axon) — context engine for AI coding agents.

> **Source code** lives in a private repository. This repo distributes compiled binaries only.

## Install

### Homebrew (recommended)

```bash
brew tap HideakiSolutions/axon
brew install axon
```

### Direct download

| Platform | Latest release |
|----------|---------------|
| Linux x86-64 | [axon-linux-x64.tar.gz](https://github.com/HideakiSolutions/axon-releases/releases/latest) |
| macOS Apple Silicon | [axon-macos-arm64.tar.gz](https://github.com/HideakiSolutions/axon-releases/releases/latest) |

```bash
VERSION=0.5.5
curl -L -o axon.tar.gz \
  "https://github.com/HideakiSolutions/axon-releases/releases/download/v${VERSION}/axon-${VERSION}-linux-x64.tar.gz"
tar xzf axon.tar.gz && cd "axon-${VERSION}-linux-x64"
./install.sh /path/to/your-project
```

### After installation

```bash
axon-setup /path/to/your-project   # wires Claude Code hooks
axon index /path/to/your-project   # index the codebase
axon serve                          # start MCP server
```

## Releases

See the [Releases page](https://github.com/HideakiSolutions/axon-releases/releases) for all versions with changelogs and SHA-256 checksums.

## License

MIT — binaries are built from the axon source under the MIT License.
