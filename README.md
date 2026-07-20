# terraform-lsp-cc-plugin

A Claude Code plugin marketplace that ships
[terraform-ls](https://github.com/hashicorp/terraform-ls) — HashiCorp's
Terraform language server — as an LSP for Claude Code.

Sibling of
[pyrefly-lsp-cc-plugin](https://github.com/michael-denyer/pyrefly-lsp-cc-plugin),
sharing the same shim pattern: memory cap, idle kill, client-driven respawn.

## Plugins

| Plugin | Description |
|--------|-------------|
| [`terraform-lsp`](./plugins/terraform-lsp/) | Registers `terraform-ls serve` as the language server for `.tf` / `.tfvars` files, with a 750 MiB memory cap and 24 h idle kill. |

## Install

```text
/plugin marketplace add michael-denyer/terraform-lsp-cc-plugin
/plugin install terraform-lsp@terraform-lsp-cc-plugin
```

The shim finds `terraform-ls` on `PATH` or falls back to the copy bundled
with the VS Code HashiCorp Terraform extension — see the plugin
[README](./plugins/terraform-lsp/README.md) for details, including why you
probably want it enabled per project rather than globally.

## License

MIT. See [LICENSE](./LICENSE).
