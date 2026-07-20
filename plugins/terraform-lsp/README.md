# terraform-lsp

Wires [terraform-ls](https://github.com/hashicorp/terraform-ls) — HashiCorp's
Terraform language server — into Claude Code's built-in LSP integration.

Follows the same pattern as
[pyrefly-lsp-cc-plugin](https://github.com/michael-denyer/pyrefly-lsp-cc-plugin),
including the memory cap.

## Prerequisites

A `terraform-ls` binary. The shim looks for one in this order:

1. `terraform-ls` on `PATH` (`brew install hashicorp/tap/terraform-ls`)
2. The copy bundled with the VS Code HashiCorp Terraform extension
   (`~/.vscode/extensions/hashicorp.terraform-*/bin/terraform-ls`)

## Intended use: enable per project, not globally

Language servers cost memory whether or not you use them. Keep the plugin
disabled at user scope and enable it only in projects that contain Terraform:

```bash
# once, from anywhere: register and install
claude plugin marketplace add <path-or-repo>
claude plugin install terraform-lsp@terraform-lsp-cc-plugin

# in each Terraform project:
claude plugin enable --scope project terraform-lsp@terraform-lsp-cc-plugin
```

`--scope project` writes to the project's `.claude/settings.json`, so the
server only runs in sessions inside that project.

## How it works

The plugin registers an LSP server pointing at a shim
([`bin/terraform-lsp`](./bin/terraform-lsp)) that:

1. Locates `terraform-ls` (PATH first, VS Code bundle as fallback).
2. If missing, exits with a clear stderr message listing install options.
3. Launches `terraform-ls serve` and watches its memory.

Handles: `.tf`, `.tfvars`.

## Memory cap

Long-lived language servers accumulate memory. The shim polls the server
every 15 minutes and kills it when it crosses **750 MiB** (default), or when
it has done no work for **24 hours** (idle detection via the server's
accumulated CPU time — a server nobody queries burns no CPU). Claude Code
respawns and re-initializes it on the next LSP request. The shim cannot
restart the server itself — the LSP client owns the `initialize` handshake —
which is why it exits and lets the client respawn.

| Variable | Default | Meaning |
|---|---|---|
| `TERRAFORM_LSP_MAX_RSS_KB` | `768000` (750 MiB) | RSS cap in KB; `0` disables the cap |
| `TERRAFORM_LSP_POLL_SECS` | `900` (15 min) | Seconds between checks |
| `TERRAFORM_LSP_IDLE_SECS` | `86400` (24 h) | Kill after this long with no CPU activity; `0` disables |

When the cap fires, the shim logs one line to stderr, visible in Claude
Code's LSP logs.

## License

MIT. See [LICENSE](./LICENSE).
