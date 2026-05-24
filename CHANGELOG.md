# Changelog

All notable changes to axon will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.1.2] — 2026-05-24

### Added

- Published Linux, macOS, and Windows installers for axon `v1.1.2`.
- Native C++ CLI now exposes `axon web` for the browser graph explorer and
  HTTP API, plus `axon lsp` for Language Server Protocol editor integration.

### Changed

- Updated release documentation and examples to use `v1.1.2` and `axon web`.
- Release assets were created from the successful build artifacts after the
  workflow publish step hit a transient GitHub API rate limit.

## [1.1.1-docs] — 2026-05-23

### Documentation

- Added release-packaging guidance clarifying that installers are generated from
  the private `HideakiSolutions/axon` source repository by
  `.github/workflows/release.yml` and published here.
- Clarified that the current tracked Linux artifact exposes `axon serve --http`
  for the web API, but does not yet expose standalone `axon web` or `axon lsp`
  commands.

## [1.1.0] — 2026-05-22

### Added

- **Dialogue Layer** — native structured conversation memory in the same DuckDB store used for the code graph. Threads, sessions, turns, anchors, and digests, all locally stored with 768-dim embeddings via nomic-embed-text.
- **Auto-anchor** — on every `turn_add`, axon scans content with a regex for source file paths (23 extensions) and performs word-boundary lookup against the top-500 most-referenced symbols in the dependency graph, automatically linking turns to code artifacts in `turn_anchors`.
- **Axon Digest Format (ADF)** — rule-based session summary generated on `session_end`: `[SESSION:]`, `[ANCHORS:]`, and turn excerpts (first, last, anchored). Embedded with nomic-embed-text for semantic retrieval.
- **`get_context_capsule` extended** — new optional `dialogue_budget` parameter: when > 0, the capsule response includes a `related_turns[]` array of past conversations anchored to the same pivot files, ranked by cosine similarity, within budget. Zero-overhead when omitted.
- **11 new MCP tools** (26 total, up from 15): `thread_create`, `thread_list`, `thread_get`, `session_start`, `session_end`, `turn_add`, `turn_search`, `session_get`, `anchor_link`, `dialogue_context`.
- **4 new HTTP REST endpoints**: `GET /api/threads`, `GET /api/threads/:id/sessions`, `GET /api/sessions/:id/turns`, `GET /api/dialogue/search`.
- **`embed_pending_turns`** — mirrors `embed_pending_symbols`; drains turns with NULL embedding on `run_pipeline`, `index_paths`, and the background sync path.
- **New DB tables**: `threads`, `sessions`, `turns`, `turn_anchors` (incremental migration, backwards-compatible).

### Changed

- **DuckDB 1.1.3 → 1.2.2** — fixes a silent bug where `UPDATE` on `FLOAT[N]` columns produced no effect.
- `run_pipeline` and `index_paths` responses now include `turns_embedded` count alongside `symbols_embedded`.

## [1.0.0] — 2026-04-19

### Added

- Multi-repo group registry (`group_list`, `group_impact`).
- Graph-assisted rename (`rename`).
- Route map and API impact analysis (`route_map`, `api_impact`).
- `detect_changes` — git-aware change tracking.
- HTTP mode with 4 REST endpoints.
- 15 MCP tools with full MCP protocol compliance.
- Write-through indexing (auto-reindex after edits).
- Hybrid search (graph BFS + semantic embeddings).
- tree-sitter grammars for 13 languages (TypeScript, JavaScript, Python, Rust, Go, C#, PHP, Dart, Java, Bash, C++, Kotlin, Vue).
