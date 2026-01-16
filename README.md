# Mise Cookbooks :cook:

Several [mise](https://github.com/jdx/mise) setups that I find useful. :sparkles:

## Usage :hammer_and_wrench:

### Use the `micoo` CLI :rocket:

You can use the `micoo` CLI to interact with the cookbooks.

List all available cookbooks:

```shell
mise exec pipx:micoo -- micoo list
```

Dump a specific cookbook to your `mise.toml` file:

```shell
mise exec pipx:micoo -- micoo dump <cookbook_name> > mise.toml
```

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
- [Zig](./zig.mise.toml) :zap: — A cookbook for managing Zig projects with the Zig build system.
- [Django](./django.mise.toml) :snake: — A cookbook for managing Django web applications.
- [Deno](./deno.mise.toml) :sauropod: — A cookbook for managing Deno projects with built-in tooling.
- [Jupyter](./jupyter.mise.toml) :notebook: — A cookbook for managing Jupyter notebooks and data science workflows.

## Contributing :heart:

If you have a useful cookbook that you would like to share, please start a [discussion](https://github.com/hasansezertasan/mise-cookbooks/discussions) or open a pull request. :octocat:
