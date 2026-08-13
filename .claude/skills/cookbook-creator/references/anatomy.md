# Cookbook Anatomy Reference

Full field reference, shared task recipes, and per-type templates for
`<name>.mise.toml` files. Read the section that matches the cookbook you're
building, then adapt the nearest existing cookbook rather than writing from
scratch.

## Table of contents

- [File skeleton](#file-skeleton)
- [Field reference](#field-reference)
- [Shared task recipes](#shared-task-recipes)
- [Templates by type](#templates-by-type) — compiled language, interpreted runtime, Python web framework, package manager
- [Conventions checklist](#conventions-checklist)

## File skeleton

Every cookbook follows this top-to-bottom order:

```toml
min_version = "2024.9.5"

[env]
PROJECT_NAME = "{{ config_root | basename }}"
# ...tech-specific env...

[tools]
# ...runtime + linters...

# optional: [hooks], [settings]

[tasks.install]
# ...

[tasks.info]
# ...(usually last)
```

## Field reference

### `min_version`

Always `"2024.9.5"` for new community cookbooks. It declares the minimum mise
version the config relies on. (Some cookbooks copied verbatim from mise's docs —
like `pnpm` — omit it; that's fine for those, but new ones should include it.)

### `[env]`

- `PROJECT_NAME = "{{ config_root | basename }}"` — **always present.** Names the
    project after its directory.
- Tech-specific vars as needed: `HOST`, `PORT`, `APP_MODULE`/`LITESTAR_APP`,
    `BUILD_DIR`, `CGO_ENABLED`, `RUST_BACKTRACE`, etc.
- `ENVIRONMENT = "{{ env.ENVIRONMENT | default(value='development') }}"` when the
    cookbook branches on environment.
- **Python projects** activate a venv automatically:
    `_.python.venv = { path = ".venv", create = true }`.
- Put executables on PATH with `_.path` when a tool installs binaries into the
    project (`_.path = ['{{config_root}}/node_modules/.bin']`, or a gemset `bin`).
- **Don't export a variable the tool manages itself.** Some frameworks pick their
    environment automatically and only respect an override if you *set* one —
    e.g. Mix runs `mix test` in `:test` unless `MIX_ENV` is already exported, so
    hardcoding `MIX_ENV = "…default(value='dev')"` silently forces tests to run in
    `:dev`. The same trap applies to `RAILS_ENV`, `NODE_ENV`, `PYTHON_ENV`, etc.
    Only add an env var the tasks genuinely need; when in doubt, leave the tool to
    manage its own environment. A superfluous env var isn't neutral — it can
    actively break `test`/`ci`.

### `[tools]`

- Pin the runtime with an env-overridable default:
    `python = "{{ get_env(name='PYTHON_VERSION', default='3.11') }}"`. Use the
    matching var name per language (`GO_VERSION`, `RUST_VERSION`, `RUBY_VERSION`,
    `NODE_VERSION`, ...). `rust` defaults to `"stable"`.
- Linters/formatters and helper CLIs **that have a mise backend** are pinned to
    `"latest"` (`golangci-lint = "latest"`, `ruff = "latest"`, `uv = "latest"`).
    Confirm the name against `mise registry` before adding it.
- **Package-manager-delivered linters do not go here.** Many ecosystems'
    canonical linters ship as project dependencies with no mise backend — credo
    (Hex), ameba (shards), eslint/prettier (npm). Those are installed by the
    `install` task and called directly in `lint`/`format`; expose their
    executables with `_.path` (e.g. `_.path = ['{{config_root}}/node_modules/.bin']`
    or a project `bin` dir) so `lint` resolves them. `ruby-on-rails` (rubocop) and
    `node` are the reference examples. When unsure whether a tool belongs in
    `[tools]`: if `mise registry` doesn't list it, it doesn't.

### Version/env template forms

Two interchangeable ways to inject a default; use each where the existing
cookbooks do:

- `get_env(name='FOO_VERSION', default='X')` — the standard form for **tool
    versions** in `[tools]` (matches every language cookbook).
- `env.FOO | default(value='X')` — the filter form used for **`[env]` values**
    like `HOST`, `PORT`, `ENVIRONMENT`, `NODE_ENV` (matches `node`, `uv`).

### `[tasks.<name>]`

- `description` — short imperative sentence. Required.
- `alias` — a one-letter shortcut where there's an obvious one: `i` install,
    `b` build, `r` run, `t` test, `l` lint, `f` format, `c` check/clean. Only add
    aliases that don't collide within the file.
- `run` — the command. Use a `'''…'''` multiline block for multi-command tasks
    (like `info`).
- `depends` — list of task names to run first. The `ci` task is **composed with
    `depends`, never a `run`**: `depends = ["lint", "test"]`.
- `sources` / `outputs` — for install-style tasks that should be cached/skipped
    when inputs are unchanged (see the package-manager template).
- Namespaced subtasks use quoted dotted keys: `[tasks."test:coverage"]`,
    `[tasks."build:release"]`.
- Hidden helper tasks use `hide = true` (rare in cookbooks; common in the repo's
    own `.config/mise.toml`).

## Shared task recipes

The canonical `info` task — adapt the fields to the technology:

```toml
[tasks.info]
description = "Print project information"
run = '''
echo "Project: $PROJECT_NAME"
echo "<Runtime>: $(<runtime> --version)"
'''
```

For Python frameworks, `info` typically also prints the venv and probes the
framework version without failing when it's not installed:

```toml
run = '''
echo "Project: $PROJECT_NAME"
echo "Virtual Environment: $VIRTUAL_ENV"
echo "Python: $(python --version)"
python -c "import <pkg>; print(f'<Pkg>: {<pkg>.__version__}')" 2>/dev/null || echo "<Pkg>: not installed"
'''
```

The canonical `ci` task is `lint` + `test`:

```toml
[tasks.ci]
description = "Run all checks (lint, test)"
depends = ["lint", "test"]
```

Extend the `depends` list when the ecosystem has an idiomatic step people
genuinely run as part of CI — most often a type/consistency check:
`["lint", "check", "test"]` (SvelteKit's `svelte-check`), or Go's
`["lint", "vet", "test"]`. Keep the `description` in sync with the list. Avoid
adding `build` to `ci` unless a sibling cookbook for that language family
already does (`nim` does; most don't) — a green lint+test is the baseline
contract.

## Templates by type

### Compiled language

Modeled on `go.mise.toml` / `rust.mise.toml`. Key traits: `build` +
`build:release`, a `clean` task, `ci` depends on check/lint/test.

```toml
min_version = "2024.9.5"

[env]
PROJECT_NAME = "{{ config_root | basename }}"
BUILD_DIR = "bin"

[tools]
<lang> = "{{ get_env(name='<LANG>_VERSION', default='X.Y') }}"
<linter> = "latest"

[tasks.build]
description = "Build the project"
alias = "b"
run = "<build command> -o $BUILD_DIR/$PROJECT_NAME"

[tasks."build:release"]
description = "Build with optimizations"
run = "<release build command>"

[tasks.run]
description = "Run the application"
alias = "r"
run = "<run command>"

[tasks.test]
description = "Run tests"
alias = "t"
run = "<test command>"

[tasks.lint]
description = "Lint the code"
alias = "l"
run = "<lint command>"

[tasks.format]
description = "Format the code"
alias = "f"
run = "<format command>"

[tasks.clean]
description = "Clean build artifacts"
alias = "c"
run = "rm -rf \"$BUILD_DIR\""

[tasks.ci]
description = "Run all checks (lint, test)"
depends = ["lint", "test"]

[tasks.info]
description = "Print project information"
run = '''
echo "Project: $PROJECT_NAME"
echo "<Lang>: $(<lang> --version)"
'''
```

### Interpreted runtime

Modeled on `node.mise.toml` / `deno.mise.toml`. Centered on `install`, `run`,
`test`, `lint`, `format`.

```toml
min_version = "2024.9.5"

[env]
PROJECT_NAME = "{{ config_root | basename }}"

[tools]
<runtime> = "{{ get_env(name='<RT>_VERSION', default='X') }}"

[tasks.install]
description = "Install dependencies"
alias = "i"
run = "<install command>"

[tasks.run]
description = "Run the application"
alias = "r"
run = "<run command>"

[tasks.test]
description = "Run tests"
alias = "t"
run = "<test command>"

[tasks.lint]
description = "Lint the code"
alias = "l"
run = "<lint command>"

[tasks.format]
description = "Format the code"
alias = "f"
run = "<format command>"

[tasks.ci]
description = "Run all checks (lint, test)"
depends = ["lint", "test"]

[tasks.info]
description = "Print project information"
run = '''
echo "Project: $PROJECT_NAME"
echo "<Runtime>: $(<runtime> --version)"
'''
```

### Python web framework

Modeled on `fastapi.mise.toml` / `litestar.mise.toml` / `flask.mise.toml`. Key
traits: auto venv, `uv`+`ruff`, a `server` (dev, `--reload`) distinct from `run`
(production), framework-specific extras.

```toml
min_version = "2024.9.5"

[env]
PROJECT_NAME = "{{ config_root | basename }}"
APP_MODULE = "app.main:app"
HOST = "127.0.0.1"
PORT = "8000"

# Automatic virtualenv activation
_.python.venv = { path = ".venv", create = true }

[tools]
python = "{{ get_env(name='PYTHON_VERSION', default='3.11') }}"
uv = "latest"
ruff = "latest"

[tasks.install]
description = "Install dependencies"
alias = "i"
run = "uv pip install -r requirements.txt"

[tasks.server]
description = "Run the development server with auto-reload"
alias = "s"
run = "<framework dev command> --reload --host $HOST --port $PORT"

[tasks.run]
description = "Run the production server"
alias = "r"
run = "<framework prod command> --host $HOST --port $PORT"

[tasks.test]
description = "Run tests"
alias = "t"
run = "pytest tests/"

[tasks."test:coverage"]
description = "Run tests with coverage"
run = "pytest tests/ --cov=app --cov-report=html"

[tasks.lint]
description = "Lint the code"
alias = "l"
run = "ruff check ."

[tasks.format]
description = "Format the code"
alias = "f"
run = "ruff format ."

[tasks.ci]
description = "Run all checks (lint, test)"
depends = ["lint", "test"]

[tasks.info]
description = "Print project information"
run = '''
echo "Project: $PROJECT_NAME"
echo "Virtual Environment: $VIRTUAL_ENV"
echo "Python: $(python --version)"
python -c "import <pkg>; print(f'<Pkg>: {<pkg>.__version__}')" 2>/dev/null || echo "<Pkg>: not installed"
'''
```

### Package manager

Modeled on `pnpm.mise.toml` / `uv.mise.toml`. Key traits: `[hooks]` for
setup, `sources`/`outputs` on the install task for caching, and tasks that
`depends` on install.

```toml
[tools]
<runtime> = 'X'

[hooks]
postinstall = '<enable/sync command>'

[env]
_.path = ['{{config_root}}/<bin dir>']

[tasks.install]
description = 'Install dependencies'
run = '<manager> install'
sources = ['<manifest>', '<lockfile>', 'mise.toml']
outputs = ['<install marker>']

[tasks.dev]
description = 'Run the dev script'
run = '<dev command>'
depends = ['install']
```

## Conventions checklist

Before running `mise run ci`, confirm:

- [ ] Filename is lowercase, hyphenated: `<tech>.mise.toml`.
- [ ] `min_version = "2024.9.5"` present.
- [ ] `[env]` defines `PROJECT_NAME`.
- [ ] Runtime pinned via `get_env(..., default=...)`; mise-backed linters at
        `"latest"`.
- [ ] Every name in `[tools]` verified against `mise registry`;
        package-manager-delivered linters kept out of `[tools]` and reached via
        `_.path`.
- [ ] Tasks have `description`; obvious ones have single-letter `alias`.
- [ ] `ci` uses `depends`, not `run` (`lint`+`test`, plus a `check` step only if
        idiomatic).
- [ ] An `info` task echoes project + version facts.
- [ ] Task `run` commands are the technology's real commands.
- [ ] README.md has a matching Community Cookbooks bullet.
