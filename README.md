# Neovim configuration

Personal Neovim configuration for Lua, Python, TypeScript, and Scala development.

## Install

Requires Neovim 0.12+, Git, Make, ripgrep, a Nerd Font, `stylua`, `scalafmt`, and tree-sitter CLI 0.26.1+.

```sh
git clone git@github.com:BnjmnZmmrmn-parafin/nvim.config.git ~/.config/nvim
nvim
```

Plugins install automatically. Mason installs `lua_ls`, `pyright`, and `ts_ls`; Metals installs through Coursier. See [`CLAUDE.md`](CLAUDE.md) for keymaps and Metals troubleshooting.
