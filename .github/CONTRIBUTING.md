# Contributing

Contributions are welcome! To share a cookbook, please start a
[discussion](https://github.com/hasansezertasan/mise-cookbooks/discussions) or
open a pull request.

## Development setup

Install the tools and git hooks with [mise](https://github.com/jdx/mise) and
[hk](https://github.com/jdx/hk):

```shell
mise install
hk install
```

## Git hooks

`hk` (configured in `hk.pkl`) runs the CI checks automatically:

- `pre-commit`: `mise run ci` with auto-fix enabled
- `pre-push`: `mise run ci`

Run `mise run ci` anytime to check + lint + format.

## Agent skills

This repo ships a `cookbook-creator` skill under `.claude/skills/` to help agents
scaffold new cookbooks. That format is currently **Claude Code-specific**.

If you use another AI coding agent (Codex, opencode, Cursor, Copilot, Gemini,
...), the repository conventions live in [`AGENTS.md`](../AGENTS.md) (mirrored by
`CLAUDE.md`), which most agents read. You can also follow the skill's guidance
directly by reading `.claude/skills/cookbook-creator/SKILL.md`.
