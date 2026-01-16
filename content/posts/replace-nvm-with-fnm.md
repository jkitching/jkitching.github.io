---
title: "Replacing nvm with fnm for faster shell startup"
date: 2026-01-16
---

If you've used [nvm](https://github.com/nvm-sh/nvm) (Node Version Manager) for any length of time, you've probably noticed the shell startup delay. On my machine, nvm adds roughly 300-500ms to every new terminal---enough to feel sluggish.

The common workaround is lazy loading. oh-my-zsh even has built-in support for it:

```zsh
zstyle ":omz:plugins:nvm" lazy yes
NVM_LAZY_LOAD=true
```

This defers nvm initialization until you actually run `node`, `npm`, or `nvm`. But lazy loading only wraps a fixed list of commands (`nvm`, `node`, `npm`, `npx`, etc.). Globally-installed binaries like `claude` aren't wrapped by default---you need to add them manually:

```zsh
zstyle ':omz:plugins:nvm' lazy-cmd claude
```

Either way, you're working around nvm's slow startup rather than solving it.

## fnm: A faster alternative

[fnm](https://github.com/Schniz/fnm) (Fast Node Manager) is a drop-in replacement written in Rust. It's a single binary with near-instant startup---no lazy loading needed.

## Installation

```bash
curl -fsSL https://fnm.vercel.app/install | bash
```

This installs fnm to `~/.local/share/fnm`.

## Shell configuration

Add to your `.zshrc`:

```zsh
FNM_PATH="$HOME/.local/share/fnm"
if [ -d "$FNM_PATH" ]; then
  export PATH="$FNM_PATH:$PATH"
  eval "$(fnm env)"
fi
```

The `fnm env` command creates a per-shell temporary directory at `/run/user/$UID/fnm_multishells/<pid>_<timestamp>/bin/` with symlinks to the active Node version. This allows different terminals to use different Node versions simultaneously.

## Install Node and restore global packages

```bash
fnm install --latest
```

Then reinstall any global packages you need:

```bash
npm install -g @anthropic-ai/claude-code
```

## Optional: auto-switch versions per directory

If you use `.nvmrc` or `.node-version` files in your projects, add `--use-on-cd`:

```zsh
eval "$(fnm env --use-on-cd)"
```

## Cleanup

Once you've verified fnm works, you can remove nvm:

```bash
rm -rf ~/.nvm
```

And remove any nvm-related lines from your `.zshrc`.
