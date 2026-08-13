# CLAUDE.md

Guidance for agents working in this repository. For the project overview and the
full cookbook list, see [README.md](./README.md); for dev setup and git hooks,
see [CONTRIBUTING.md](./.github/CONTRIBUTING.md).

## Cookbook Structure

Each cookbook (`*.mise.toml`) follows a consistent pattern:

1. **`min_version`** - Minimum mise version required
2. **`[env]`** - Environment variables (PROJECT_NAME, framework-specific vars)
3. **`[tools]`** - Runtime version + linting tools
4. **`[tasks]`** - Common commands: `install`, `run`, `test`, `lint`, `format`, `ci`, `info`

## Checks

Run `mise tasks ls --all --hidden` to discover tasks. Run `mise run ci` before
committing — it runs the same check + lint + format as CI.
