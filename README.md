# Mise Cookbooks :cook:

Several [mise](https://github.com/jdx/mise) setups that I find useful. :sparkles:

## Usage :hammer_and_wrench:

### Use the `cobo` CLI :rocket:

You can use the [`cobo`](https://github.com/hasansezertasan/cobo) CLI to interact with the cookbooks.

List all available cookbooks:

```shell
mise exec pipx:cobo -- cobo mise list
```

Dump a specific cookbook to a file:

```shell
mise exec pipx:cobo -- cobo mise dump <cookbook_name> > mise.local.toml
```

> [!NOTE]
> Redirecting to `mise.local.toml` avoids truncating an existing `mise.toml`.

### Haters gonna hate but still useful :sunglasses:

Just copy and paste the contents of the technology-specific `.mise.toml` file into your own `mise.toml` file. :clipboard:

## Cookbooks :books:

### Official Cookbooks :star:

> These cookbooks are taken from the official mise documentation. :bookmark_tabs:

- [terraform](./terraform.mise.toml) :earth_africa: — A cookbook for managing Terraform configurations using the official Terraform CLI.
- [C++](./cpp.mise.toml) :heavy_plus_sign: — A cookbook for managing C++ projects, including build systems and dependencies.
- [Node](./node.mise.toml) :deciduous_tree: — A cookbook for managing Node.js projects, including package management and scripts.
- [pnpm](./pnpm.mise.toml) :package: — A cookbook for managing projects using pnpm, a fast and disk space-efficient package manager.
- [Python](./python.mise.toml) :snake: — A cookbook for managing Python projects, including virtual environments.
- [Ruby on Rails](./ruby-on-rails.mise.toml) :gem: — A cookbook for managing Ruby on Rails applications, including gems and database migrations.

### Community Cookbooks :handshake:

- [opentofu](./opentofu.mise.toml) :seedling: — A cookbook for managing Terraform configurations using OpenTofu.
- [Go](./go.mise.toml) :rocket: — A cookbook for managing Go projects with golangci-lint.
- [Rust](./rust.mise.toml) :crab: — A cookbook for managing Rust projects with Cargo and Clippy.
- [Zig](./zig.mise.toml) :high_voltage: — A cookbook for managing Zig projects with the Zig build system.
- [Django](./django.mise.toml) :snake: — A cookbook for managing Django web applications.
- [Deno](./deno.mise.toml) :sauropod: — A cookbook for managing Deno projects with built-in tooling.
- [Jupyter](./jupyter.mise.toml) :notebook: — A cookbook for managing Jupyter notebooks and data science workflows.
- [Bun](./bun.mise.toml) :bread: — A cookbook for managing Bun projects with built-in bundler and test runner.
- [FastAPI](./fastapi.mise.toml) :zap: — A cookbook for managing FastAPI applications with uvicorn.
- [Flask](./flask.mise.toml) :coffee: — A cookbook for managing Flask web applications.
- [Litestar](./litestar.mise.toml) :star2: — A cookbook for managing Litestar applications with the Litestar CLI.
- [PHP](./php.mise.toml) :elephant: — A cookbook for managing PHP projects with Composer and Mago.
- [Nim](./nim.mise.toml) :lemon: — A cookbook for managing Nim projects with Nimble.
- [Ruby Gemset](./ruby-gemset.mise.toml) :gem: — A cookbook for RVM-style, project-local gemset isolation in Ruby projects.

## Contributing :heart:

If you have a useful cookbook that you would like to share, please start a [discussion](https://github.com/hasansezertasan/mise-cookbooks/discussions) or open a pull request. :octocat: See [CONTRIBUTING.md](./.github/CONTRIBUTING.md) for dev setup and git hooks.
