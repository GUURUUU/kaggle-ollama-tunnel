# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning is [SemVer](https://semver.org/).

## [1.0.0] - 2026-11-08

First public release.

### Added
- Single-cell Kaggle notebook: installs Ollama, pulls an F16 abliterated GGUF
  (Llama-3.1-8B variant, swap-friendly via `REPO_CANDIDATES`), imports it,
  verifies the local OpenAI-compatible endpoints on `:11434`.
- Cloudflare Quick Tunnel exposure with the `--http-host-header` trick (no 403s).
- 5-minute keep-alive loop: VRAM pinning via `keep_alive: -1`, tunnel restart,
  Ollama crash recovery after 3 failed heartbeats.
- CI: notebook validation (nbformat + AST parse) and a secret scan on every push.
- Docs: README with client examples (Python/OpenAI SDK + curl), demo GIF,
  social preview card; CONTRIBUTING guide; issue templates.
