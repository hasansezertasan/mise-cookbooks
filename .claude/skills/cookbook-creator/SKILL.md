---
name: cookbook-creator
description: >-
    Create a new mise cookbook (a `<name>.mise.toml`) for a language, framework,
    stack, package manager, or other technology in the mise-cookbooks repo. Use
    this whenever the user wants to add, generate, scaffold, or write a cookbook —
    e.g. "add a cookbook for Elixir", "make a Svelte cookbook", "we should have
    one for Gleam", "scaffold a mise setup for Kotlin" — even if they don't say the
    word "cookbook". Handles researching the real toolchain, writing the toml to
    the repo's exact conventions, registering it in README.md, and verifying it
    with `mise run ci`.
---

# Cookbook Creator

A cookbook is a single `<name>.mise.toml` at the repo root that gives a project
a ready-made mise setup for one technology: pinned tool versions, sensible env
vars, and a consistent set of tasks (`install`, `run`, `test`, `lint`,
`format`, `ci`, `info`). Someone drops it into their project as `mise.toml` and
immediately has `mise run test`, `mise run lint`, and friends.

The value of a cookbook is that it *actually works* — the task commands invoke
the real tools with the real flags. So the job is half research (what are this
technology's genuine install/run/test/lint/format commands, and what does mise
call the tool?) and half fitting the repo's house style. Get the commands wrong
and the cookbook is worse than nothing.

## Workflow

Do these in order. Don't skip the research step — guessing tool names and CLI
flags is the most common way these go wrong.

### 1. Research the real toolchain

Before writing anything, pin down the facts. You need:

- **The mise tool name(s).** Run `mise registry | grep -i <tech>` to find how
    mise refers to the tool (e.g. `go`, `rust`, `bun`, `deno`, `zig`). Not every
    tool is a first-class mise plugin — some come via `pipx:`, `npm:`, `cargo:`,
    `aqua:`, or `ubi:` backends. Verify before you commit to a name.
- **The genuine commands** for install, run/build, test, lint, and format. When
    unsure, consult the technology's official docs — a web search, or any docs/MCP
    tooling available in your session (e.g. a context7 server if one is
    connected). Don't rely on memory for CLI flags; they drift between versions.
- **The idiomatic linter/formatter, and how it's delivered.** Most ecosystems
    have a canonical choice (Go → golangci-lint + `go fmt`, Rust → clippy +
    `cargo fmt`, Python → ruff, Ruby → rubocop, Elixir → credo, Crystal → ameba,
    JS/TS → eslint + prettier). Prefer it over exotic alternatives. Then decide
    **where it comes from** — two valid patterns, so mirror the closest sibling
    cookbook. Put it **in `[tools]`** if mise can install it, which means a
    registry name *or* an explicit backend spec: `node.mise.toml` pins
    `"npm:eslint"` and `"npm:typescript"` that way, and `cargo:`, `pipx:`,
    `aqua:`, `ubi:` work the same. Keep it **out of `[tools]`** when the linter is
    just another manifest dependency (credo in `mix.exs`, ameba in `shard.yml`) or
    you'd rather run it from the project's own `node_modules`; then the `install`
    task pulls it in and `lint`/`format` call it directly, with its bin dir on PATH
    via `_.path` (see how `ruby-on-rails` runs rubocop). A missing `mise registry`
    entry alone doesn't force this second route — check whether a backend can
    install it first.

If the target is a **framework** rather than a bare language (FastAPI, Django,
Rails, Litestar), find the framework's own dev-server / management commands and
build on top of the base language's cookbook conventions.

### 2. Classify the cookbook

The task set depends on what kind of technology this is. See
`references/anatomy.md` for the full patterns, but roughly:

- **Compiled language** (Go, Rust, Zig, Nim): `build` + `build:release`, `run`,
    `test`, `lint`, `format`, `clean`, `ci`, `info`.
- **Interpreted language / runtime** (Node, Deno, Python, Bun): `install`,
    `run`, `test`, `lint`, `format`, `ci`, `info`.
- **Web framework** (FastAPI, Flask, Django, Litestar, Rails): add `server`
    (dev, auto-reload) vs `run` (production), plus framework specials (`routes`,
    `openapi`, `migrate`, ...).
- **Package manager / meta-tool** (pnpm, uv): center on the manager's own
    workflow, often with `[hooks]` and `sources`/`outputs` for caching.

Pick the closest existing cookbook as your template and adapt it — consistency
with siblings matters more than inventing new structure.

### 3. Write `<name>.mise.toml`

Follow the repo's exact conventions. The essentials, in order:

1. `min_version = "2024.9.5"` (unless copying an official cookbook that omits it).
2. `[env]` — always define `PROJECT_NAME = "{{ config_root | basename }}"`;
    add framework/tool env vars (host, port, app module, build dir) as needed.
3. `[tools]` — pin the runtime with an env-overridable default, using whichever
    form the closest sibling cookbook uses:
    `tool = "{{ get_env(name='FOO_VERSION', default='X.Y') }}"` or the equivalent
    `tool = "{{ env['FOO_VERSION'] | default(value='X.Y') }}"` (as `node` does).
    Add helper tools mise can install as `"latest"` — including explicit backend
    specs like `"npm:eslint"`. Only deps the language's own package manager
    resolves from its manifest (credo, ameba) stay out (see step 1).
4. `[tasks.*]` — each task has `description`, an `alias` where there's an obvious
    short form (`i` `b` `r` `t` `l` `f` `c`), and `run`. The `ci` task uses
    `depends = [...]`, not `run` — usually `["lint", "test"]`; fold in a
    type-check step (`check`) when the ecosystem has one people actually run
    (e.g. `svelte-check`, `mix compile --warnings-as-errors`). Don't put `build`
    in `ci` unless a sibling cookbook for that family already does. The `info`
    task echoes project + version facts.

Read `references/anatomy.md` for the complete field reference, the shared task
recipes, and copy-paste-ready templates for each cookbook type.

Name the file after the technology in lowercase, hyphenated
(`ruby-on-rails.mise.toml`, `ruby-gemset.mise.toml`).

### 4. Register it in README.md

Add one bullet to the **Community Cookbooks** section (new contributions go
there; the "Official" section is reserved for cookbooks lifted verbatim from
mise's own docs). Match the existing line format exactly:

```markdown
- [Name](./name.mise.toml) :emoji: — A cookbook for managing <tech> projects with <key tools>.
```

Pick a fitting GitHub emoji shortcode (`:crab:` Rust, `:snake:` Python, ...) and
keep the em-dash (`—`) and phrasing consistent with neighbors.

### 5. Verify with `mise run ci`

Run `mise run ci` from the repo root. This runs the repo's checks, linters, and
formatters (editorconfig, typos, yamllint, markdownlint, taplo for TOML, plus
`mise fmt`). It will reformat your TOML and flag typos or style issues — fix
anything it reports and re-run until clean.

If it fails immediately with an untrusted-config error (common on a fresh
checkout or worktree), run `mise trust` first — mise walks up to the repo root,
so you may need to trust both the local `.config/mise.toml` and the root one —
then re-run `mise run ci`. This is the same gate CI and the
pre-commit hook enforce, so a green `mise run ci` means the cookbook is
mergeable.

Note: `mise run ci` validates *this repo's* formatting and does not execute the
cookbook's own tasks (that would need the target toolchain installed). Sanity-
check the task commands against your research instead — and if the target tool
happens to be installable, `mise tasks ls` in a scratch dir using the cookbook
is a nice extra confirmation.

The cookbook `.mise.toml` is the only file you normally touch, and taplo keeps it
clean automatically. If you also edit Markdown here (this skill's own docs, or
the README), mind that the repo runs two linters with a subtle conflict: the
`.editorconfig` demands every line's indent be a multiple of **4**, while
markdownlint auto-fixes nested list markers back to **2**. They fight over nested
bullet lists. Sidestep it — keep Markdown lists flat (no nested bullets; inline
sub-items on one line), and indent any wrapped continuation lines to 4 spaces.

## Quality bar

- **Commands must be real.** Every `run` should be something you'd actually type
    in a project using that technology. When in doubt, cite the official docs.
- **Consistency over cleverness.** A new cookbook should look like it was
    written by the same person who wrote the other 20. Mirror their task names,
    aliases, env vars, and `info` output style.
- **Don't over-scope.** The common task set covers most needs. Add specials only
    when they're genuinely idiomatic for the technology (a `migrate` for a web
    framework, `doc` for Rust). Resist inventing tasks nobody would run. The same
    restraint applies to `[env]`: don't set a variable the tool manages on its own
    (e.g. `MIX_ENV`, `RAILS_ENV`, `NODE_ENV`) — a needless env var can silently
    break `test`/`ci` by overriding the environment the tool would have picked.
