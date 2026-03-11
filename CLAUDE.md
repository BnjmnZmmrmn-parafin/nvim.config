# Neovim Configuration

Fork of [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim), trimmed down to a single `init.lua`.

## Structure

```
init.lua              -- Entire config: options, keymaps, autocommands, and plugins
lua/kickstart/        -- Bundled optional plugins (none currently enabled)
lua/custom/plugins/   -- User plugin directory (empty, import commented out)
```

## Options

| Setting | Value | Notes |
|---------|-------|-------|
| Leader | `<Space>` | Both leader and localleader |
| Nerd Font | enabled | Icons throughout UI |
| Line numbers | relative | `number` + `relativenumber` |
| Scrolloff | 20 | Generous cursor padding |
| Clipboard | `unnamedplus` | Synced with OS, deferred for startup perf |
| Undo | persistent | `undofile = true` |
| Search | smart-case | Case-insensitive unless uppercase present |

## Keymaps

| Key | Mode | Action |
|-----|------|--------|
| `<Esc>` | n | Clear search highlight |
| `<leader>q` | n | Open diagnostic quickfix list |
| `<Esc><Esc>` | t | Exit terminal mode |
| `<C-h/j/k/l>` | n | Navigate splits |
| `<leader>f` | n | Format buffer (conform.nvim) |
| `<leader>th` | n | Toggle inlay hints (LSP) |
| `<leader>s*` | n | Telescope search (`sf` files, `sg` grep, `sh` help, `sk` keymaps, etc.) |
| `<leader>/` | n | Fuzzy find in current buffer |
| `<leader><leader>` | n | Find open buffers |
| `gr*` | n | LSP actions (`grd` definition, `grr` references, `grn` rename, `gra` code action, etc.) |

## Plugins

### Core
- **lazy.nvim** — Plugin manager
- **vim-sleuth** — Auto-detect indent settings

### Navigation & UI
- **telescope.nvim** — Fuzzy finder (files, grep, LSP, buffers) with fzf-native and ui-select extensions
- **which-key.nvim** — Shows pending keybinds (0ms delay)
- **todo-comments.nvim** — Highlights TODO/FIXME/etc in comments

### LSP
- **nvim-lspconfig** + **mason.nvim** — LSP management
  - `lua_ls` — Lua
  - `pyright` — Python
  - `ts_ls` — TypeScript/JavaScript
- **nvim-metals** — Scala/sbt/Java (standalone, not managed by Mason)
- **fidget.nvim** — LSP progress indicator
- **lazydev.nvim** — Neovim Lua API completions

### Completion & Snippets
- **blink.cmp** — Autocompletion (sources: LSP, path, snippets, lazydev)
- **LuaSnip** — Snippet engine

### Formatting
- **conform.nvim** — Format on save
  - Lua → `stylua`
  - Scala → `scalafmt`

### Git
- **gitsigns.nvim** — Gutter signs for git changes

### Syntax
- **nvim-treesitter** — Syntax highlighting and indentation
  - Installed: bash, c, diff, html, lua, luadoc, markdown, query, vim, vimdoc, scala, typescript, tsx, javascript
  - `auto_install = true` for unlisted languages

### Appearance
- **solarized.nvim** — Solarized Light colorscheme
- **nvim-web-devicons** — File type icons (Nerd Font)

## Troubleshooting Metals (Scala LSP)

Metals runs outside Mason as a standalone server via `nvim-metals`. It manages its own installation and BSP connection to sbt/Bloop. When things go wrong, it's almost always one of these categories.

### Metals is stuck "Compiling" on a specific module

This usually means the build server (Bloop) has a stale or broken compilation state for that module.

1. **Check `:MetalsInfo`** — see which build target is stuck and whether Metals is actually connected to the build server.
2. **Clean the stuck module from sbt**, then reimport:
   ```
   # In a terminal (not nvim)
   sbt "project <module-name>" clean compile
   ```
3. **`:MetalsDisconnectBuildServer`** then **`:MetalsConnectBuildServer`** — forces Metals to re-establish the BSP connection without restarting everything.
4. **Nuclear option:** `:MetalsResetWorkspace` — deletes `.metals/` and `.bloop/` and re-indexes. This is slow on large monorepos but fixes most stuck states.
5. **Check `.bloop/<module>.json`** — if the JSON is malformed or points to stale JARs, Bloop can't compile. `sbt bloopInstall` regenerates these.

### "Go to definition" / "Find references" returns nothing or wrong results

Metals' symbol index can get out of sync with the actual compiled bytecode.

1. **`:MetalsCascadeCompile`** — recompiles the current file and everything that depends on it. Faster than a full rebuild and often fixes stale references.
2. **`:MetalsResetWorkspace`** — if cascade compile didn't help, nuke the index.
3. **Check if the target file is in a different module** — Metals needs that module to be compiled. If you haven't touched it, its index may not exist yet. Compile it explicitly via sbt.
4. **Look at fidget.nvim progress** — if indexing is still in progress, references won't work yet. Wait for it to finish.

### Metals won't start or attach

1. **`:MetalsInstall`** — may need to run this after a Metals version bump.
2. **Check `:messages`** for JVM errors — Metals needs a JDK (not just JRE). Verify `java -version` shows a JDK 11+.
3. **Check `.jvmopts` or `.sbtopts`** at repo root — bad JVM flags (especially memory limits) can cause Metals to OOM on startup in large repos.
4. **Verify `build.sbt` is parseable** — Metals won't connect if sbt can't load the build definition.

### Diagnostics are stale or missing

1. **`:MetalsCascadeCompile`** — the go-to for refreshing diagnostics.
2. **Save the file** — Metals only re-diagnoses on save by default, not on every keystroke.
3. **Check if the file is in a source root** — files outside `src/main/scala` or `src/test/scala` (or whatever your build defines) won't get diagnostics.

### Useful Metals commands

| Command | What it does |
|---------|--------------|
| `:MetalsInfo` | Show server status, build target info |
| `:MetalsCascadeCompile` | Recompile current file + dependents |
| `:MetalsResetWorkspace` | Delete `.metals/` + `.bloop/`, full re-index |
| `:MetalsDisconnectBuildServer` | Disconnect from Bloop/sbt BSP |
| `:MetalsConnectBuildServer` | Reconnect to build server |
| `:MetalsInstall` | Install/update Metals binary |
| `:MetalsRestartServer` | Restart without losing workspace state |
| `:MetalsLogs` | Open the Metals log (great for cryptic errors) |

### Log files

- **Metals log:** `:MetalsLogs` or `~/.cache/nvim/metals/metals.log`
- **Bloop log:** `~/.bloop/bloop.log`

When filing bugs or asking for help, grab the relevant section from these logs first.

## Not Enabled (available in `lua/kickstart/plugins/`)

- `debug` — DAP debugging
- `indent_line` — Indent guides
- `lint` — Linting (nvim-lint)
- `autopairs` — Auto bracket pairing
- `neo-tree` — File explorer
- `gitsigns` — Extended gitsigns keymaps
