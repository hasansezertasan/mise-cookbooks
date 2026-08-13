# Development Journal

A chronological record of decisions, attempts (including failures), and outcomes for this project.

---

## 2026-01-18: Project Analysis

### Context

Initial codebase analysis to understand the project structure and create documentation.

### What Was Learned

- **Project Purpose**: Collection of reusable mise configuration templates for various tech stacks
- **Current Cookbooks**: 16 total covering Python, Go, Rust, JS runtimes, web frameworks, and infrastructure tools
- **Tooling Stack**: Uses hk for git hooks, mise for task running, and multiple linters (yamllint, taplo, markdownlint-cli2, actionlint)
- **CI**: Simple GitHub Actions workflow that runs `mise ci`

### Files Created

- `CLAUDE.md` - Guidance for Claude Code instances
- `JOURNAL.md` - This development journal

---

## 2026-08-13: Declutter Repo Root

### Motivation

Root was crowded with loose tooling config dotfiles mixed in among the
`*.mise.toml` cookbooks (the product). Applied the same relocation pattern used
in `hasansezertasan/homebrew-tap#33` and `hasansezertasan/olink#28`.

### What Was Done

Relocated loose tooling config into `.config/` and `.github/`.

- `.config/`: `.markdownlint-cli2.yaml`, `.yamlfmt.yml`, `.yamllint.yaml`, `mado.toml`, `.prettierrc.yaml`, plus `mise.toml` + `mise.lock`.
- The three invoked linters do not auto-discover `.config/`, so their tasks now pass explicit config flags (`--config` / `-conf` / `-c`); `mado`/`prettier` have no task, so they were relocated only.
- mise auto-discovers `.config/mise.toml`, keeping the repo as project root and disambiguating it from the `*.mise.toml` cookbook templates.
- `.github/`: `renovate.json` (Renovate auto-discovers it there).
- Kept at root: the `*.mise.toml` cookbooks, `hk.pkl` (hooks entry point), `.editorconfig`, `.gitignore`, `LICENSE`, and the Markdown docs.

### Outcome

Root dropped from 34 to 26 tracked entries. `mise run ci` passes; every
relocated config loads from its new path (grep-verified no stale references).

---

<!-- Add new entries above this line -->
