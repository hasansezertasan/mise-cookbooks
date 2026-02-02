# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A collection of reusable [mise](https://github.com/jdx/mise) configuration templates ("cookbooks") for various programming languages and frameworks. Each `.mise.toml` file provides tool versions, environment variables, and common tasks for a specific technology stack.

## Development Commands

All commands use mise tasks defined in `mise.toml`:

```shell
mise run ci        # Run all checks (check + lint + format) - same as CI
mise run check     # Run editorconfig-checker, action-validator, typos
mise run lint      # Run yamllint, actionlint, markdownlint-cli2, taplo lint
mise run format    # Run mise fmt, yamlfmt, taplo format
```

Individual checks:
```shell
mise run check:typos              # Spell check
mise run lint:yaml                # YAML linting
mise run lint:toml                # TOML linting
mise run lint:markdownlint-cli2   # Markdown linting
mise run lint:gha                 # GitHub Actions linting
```

## Git Hooks

This repo uses [hk](https://github.com/jdx/hk) for git hooks (configured in `hk.pkl`):
- `pre-commit`: Runs `mise run ci` with auto-fix enabled
- `pre-push`: Runs `mise run ci`

To install hooks: `hk install`

## Architecture

### Cookbook Structure

Each cookbook (`*.mise.toml`) follows a consistent pattern:

1. **`min_version`** - Minimum mise version required
2. **`[env]`** - Environment variables (PROJECT_NAME, framework-specific vars)
3. **`[tools]`** - Runtime version + linting tools
4. **`[tasks]`** - Common commands: `install`, `run`, `test`, `lint`, `format`, `ci`, `info`

### Cookbook Categories

| Type | Files |
|------|-------|
| Official (from mise docs) | `terraform`, `cpp`, `node`, `pnpm`, `python`, `ruby-on-rails` |
| Community | `opentofu`, `go`, `rust`, `zig`, `django`, `deno`, `jupyter`, `bun`, `fastapi`, `flask` |

### Usage via micoo CLI

```shell
mise exec pipx:micoo -- micoo list                    # List cookbooks
mise exec pipx:micoo -- micoo dump <name> > mise.toml # Export cookbook
```

## File Conventions

- **TOML**: 2-space indent
- **YAML**: 2-space indent, max 120 char lines
- **All files**: UTF-8, LF line endings, final newline

## Development Journal

See `JOURNAL.md` for a chronological record of decisions, attempts (including failures), and outcomes. Update it when making significant changes.
