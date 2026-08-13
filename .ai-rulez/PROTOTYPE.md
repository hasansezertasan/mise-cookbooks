# ai-rulez prototype (evaluation only)

This branch is a **prototype** to evaluate [ai-rulez](https://github.com/Goldziher/ai-rulez) as a way to serve our agent skills to contributors who use tools other than Claude Code (Codex, opencode, Cursor, Copilot, Gemini, ...). It is not meant to be merged as-is — it exists so we can judge the approach against a real diff.

## The problem it addresses

PR #28 added the `cookbook-creator` skill under `.claude/skills/`. That format is Claude Code-specific, so a contributor on Codex, opencode, or Cursor gets nothing. We already hand-maintain `CLAUDE.md` **and** an identical `AGENTS.md`, so our *instruction files* are cross-agent — but our *skills* are not. ai-rulez turns one source of truth into per-tool native configs.

## What this branch contains

Only the source of truth, under `.ai-rulez/`:

- `config.toml` — project name and which tools to generate for (`presets`).
- `skills/cookbook-creator/` — the skill, ported verbatim from `.claude/skills/` (including `references/anatomy.md`).
- `context/repo-guidance.md` — the body of `CLAUDE.md`/`AGENTS.md`, migrated so those files can be regenerated instead of hand-kept.

Generated outputs are **not** committed — they are build artifacts. Regenerate with `npx ai-rulez@latest generate`. That writes native files for every preset (`.claude/skills/`, `.codex/skills/` + `AGENTS.md`, `.opencode/skills/`, `.cursor/`, `.github/copilot-instructions.md`, `.gemini/` + `GEMINI.md`) and rewrites `CLAUDE.md`. It also appends a managed block to `.gitignore` so those artifacts stay ignored.

## Findings — the cost side (why this needs a decision, not a merge)

ai-rulez is capable (skills, agents, commands, 20+ tools, MCP server), but it is **invasive**. Concrete frictions we hit in this repo:

- **It seizes `CLAUDE.md` and `AGENTS.md`.** `generate` overwrites them with files stamped `DO NOT EDIT`; our hand-written content survives only because we migrated it into `context/`. Adoption means those files stop being hand-edited.
- **`.agents/` collision with Flow.** ai-rulez generates into `.agents/` and gitignores the whole directory. Flow also roots at `.agents/`. Not tracked here today, so no damage — but a latent conflict.
- **Generated JSON breaks our CI.** `.gemini/settings.json` and the internal manifest use 2-space indent; our `.editorconfig` wants a multiple of 4, so `mise run ci` goes red whenever generated artifacts sit in the tree.
- **`ai-rulez clean` is a footgun.** It treats our hand-written `.claude/skills/cookbook-creator/` as generated and offers to delete it.
- **New build dependency.** A repo whose identity is "drop in one `mise.toml`, zero tooling" would gain a Node/Go generator in its dev loop.

## Recommendation

For a **single** skill plus already-mirrored instruction files, ai-rulez is more machinery than the problem needs. Options, cheapest first:

- **Keep the status quo (`AGENTS.md`).** Instruction files already reach 30+ agents; skills stay Claude-only until a contributor actually asks otherwise.
- **Adopt ai-rulez** if the skill/agent surface grows (several skills times several target tools). The payoff scales with N — revisit then.

Judge the `.ai-rulez/` source model here, run `ai-rulez generate` locally to see the full output footprint, and decide. Nothing here is load-bearing yet.
