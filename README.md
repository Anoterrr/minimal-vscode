# VSCode Config — Rust & Python (LazyVim-inspired)

A minimal, keyboard-driven VSCode setup tuned for **Rust** and **Python** development. The visual philosophy mirrors [LazyVim](https://lazyvim.org): remove every UI element that doesn't carry information, keep the editor surface clean, and let LSP do the heavy lifting.

---

## Requirements

| Tool | Purpose |
|---|---|
| [JetBrains Mono](https://www.jetbrains.com/lp/mono/) | Editor font with ligatures |
| [JetBrainsMono Nerd Font](https://www.nerdfonts.com/) | Terminal font with icons |

### Required Extensions

| Extension | ID | Role |
|---|---|---|
| rust-analyzer | `rust-lang.rust-analyzer` | Rust LSP |
| Ruff | `charliermarsh.ruff` | Python formatter + linter |
| Pylance | `ms-python.python` | Python type checking |
| Material Icon Theme | `pkief.material-icon-theme` | File icons |
| Tokyo Night | `enkia.tokyo-night` | Color theme |

### Optional — Vim keybindings

| Extension | ID | Role |
|---|---|---|
| VSCodeVim | `vscodevim.vim` | Modal editing + LazyVim-like shortcuts |

---

## Installation

```bash
# Clone or copy settings.json to your VSCode user config directory

# Linux / macOS
~/.config/Code/User/settings.json

# Windows
%APPDATA%\Code\User\settings.json
```

---

## Why These Decisions

### Interface

**Activity bar, tabs, status bar, scrollbar, minimap, breadcrumbs — all hidden.**
Each of these is a permanent UI fixture that consumes space but requires deliberate interaction to be useful. In a keyboard-driven workflow, you navigate via fuzzy find (`Ctrl+P`), the command palette (`Ctrl+Shift+P`), and LSP jumps (`gd`, `gr`). The sidebar is toggled on demand. Removing static chrome increases the vertical code surface without losing any capability.

**`editor.inlayHints.enabled: "offUnlessPressed"`**
Inlay hints in Rust are verbose by default — every binding gets a type annotation, every chain gets its return type. This creates visual noise that competes with the actual code. With this setting, hints appear only when you hold `Ctrl+Alt`, mirroring LazyVim's on-demand hint behavior.

**`editor.rulers: [80, 120]`**
Two soft limits: 80 chars for readability in split-pane view, 120 as the hard ceiling aligned with `rustfmt` defaults. Neither enforces wrapping — they are visual guides only.

---

### Rust

**`rust-analyzer.check.command: "clippy"`**
`cargo check` only validates compilation. Clippy catches idiomatic issues: unnecessary clones, suboptimal iterator chains, missing `#[must_use]`, and hundreds of other lints that `check` silently ignores. The trade-off is slightly higher save latency on large projects with heavy proc-macros — acceptable in most cases.

**`rust-analyzer.cargo.features: "all"`**
Analyzes all feature-gated code simultaneously. Without this, code behind a disabled feature flag appears as dead code and loses type-checking. The downside: higher memory usage on crates with many mutually exclusive features.

**`rust-analyzer.inlayHints.parameterHints.enable: false`**
Parameter name hints clutter simple function calls where the argument names are already self-documenting (e.g., `Vec::with_capacity(1024)`). Type hints and chaining hints are kept — they provide genuine signal that the source text doesn't repeat.

**`rust-analyzer.lens.implementations.enable: false`**
The implementations lens duplicates what `gr` (Go to References) already does, with worse ergonomics. It adds a persistent code lens above every trait and struct, which increases visual noise proportional to the size of the codebase.

---

### Python

**Ruff instead of Black + isort**
Ruff reimplements both formatters in Rust. It is 10–100x faster, handles import sorting natively, and is configured through a single `ruff.toml` or `pyproject.toml` entry. If your CI pipeline already enforces `black`, configure Ruff as the linter only and keep `black` as the formatter to avoid formatting disagreements between local and CI.

**`python.analysis.typeCheckingMode: "standard"` instead of `"basic"`**
`"basic"` mode skips many practical error categories — unbound variables behind Optional chains, missing attributes on narrowed types, and incorrect return types in certain patterns. `"standard"` catches these without the friction of `"strict"`, which requires full annotation coverage to be useful.

---

### File Nesting

Nesting groups related files under their primary entry point, collapsing project root noise:

- `Cargo.toml` collects `Cargo.lock`, `rust-toolchain.toml`, `rustfmt.toml`, `clippy.toml`
- `pyproject.toml` collects `uv.lock`, `poetry.lock`, `.python-version`, `ruff.toml`, cache dirs
- `.env` collects all `.env.*` variants

This does not hide files — they are accessible by expanding the parent. It reduces the cognitive load of scanning a project root with 15+ config files.

---

### Watcher & Search Exclusions

Rust's `target/` directory can contain hundreds of thousands of files after a full build. Without explicit exclusion from the file watcher, VSCode consumes significant CPU monitoring artifact churn during compilation. `target/` is excluded from the watcher and search index but kept visible in the Explorer — you may need to inspect build outputs.

---

## Vim Keybindings (Optional)

> If you prefer modal editing with LazyVim-style shortcuts, install **VSCodeVim** (`vscodevim.vim`) and uncomment the Vim section in `settings.json`.

The commented block includes:

**Leader key: `Space`** — mirrors LazyVim's default.

| Shortcut | Action |
|---|---|
| `<leader>e` | Toggle file explorer |
| `<leader>ff` | Fuzzy find files |
| `<leader>fg` | Search in files |
| `<leader>fr` | Recent files |
| `<leader>bd` | Close current buffer |
| `<leader>bo` | Close other buffers |
| `<leader>ca` | Code actions (quick fix) |
| `<leader>cr` | Rename symbol |
| `<leader>cd` | Open Problems panel |
| `<leader>mf` | Format document |
| `<leader>tt` | Toggle terminal |
| `<leader>h` | Clear search highlight |
| `gd` | Go to definition |
| `gr` | Go to references |
| `gi` | Go to implementation |
| `gh` | Hover documentation |
| `]d` / `[d` | Next / previous diagnostic |

Plugins enabled when VSCodeVim is active:
- **EasyMotion** — jump to any visible character with `<leader><leader>` prefix
- **Sneak** — two-character search motion (`s{char}{char}`)
- **Surround** — `ys`, `cs`, `ds` for adding/changing/deleting surrounding pairs

To activate, uncomment the `// ─── Vim (VSCodeVim) ───` block in `settings.json` and reload the window.

---

## Trade-offs and Known Issues

| Decision | Gain | Cost |
|---|---|---|
| Clippy on save | Catches idiomatic issues early | Higher latency on large crates with proc-macros |
| `features: "all"` in rust-analyzer | Full analysis across feature flags | Higher memory usage |
| Ruff as formatter | Speed, single tool | Conflicts with existing `black`-based CI |
| `typeCheckingMode: "standard"` | Catches real bugs | May surface errors in unannotated legacy code |
| Inlay hints off by default | Clean editor surface | Requires manual toggle to inspect types |
| `security.workspace.trust.enabled: false` | Removes trust prompt noise | Runs all workspace settings without confirmation |
