# dotfiles-mac-nix

This is my personal Mac setup, forked from [kunchenguid/dotfiles-mac-nix](https://github.com/kunchenguid/dotfiles-mac-nix). Unlike the upstream repo — which is deliberately trimmed down to be a shareable starter template — this fork is not meant for other people to use. It exists so I can continuously track how my machine and, especially, my agentic engineering workflow are configured, and keep evolving both over time: new tools, new agent skills, new personal preferences, all committed as they change.

It is built with [Nix](https://nixos.org/), [`nix-darwin`](https://github.com/nix-darwin/nix-darwin), [Home Manager](https://github.com/nix-community/home-manager), and declarative [Homebrew](https://brew.sh/). For the reasoning behind the original template's design, see the [blog post](https://open.substack.com/pub/kunchenguid/p/how-i-built-a-reproducible-mac-setup?utm_campaign=post-expanded-share&utm_medium=web) — most of it still applies here, minus the "keep it generic" constraint.

## What this repo does

It manages my Mac setup as code:

- bootstrap a fresh Mac with `setup/mac.sh`
- configure macOS defaults with `nix-darwin`
- manage user packages and shell behavior with Home Manager
- install GUI apps and macOS-native tools declaratively with Homebrew
- keep app config, editor config, and other personal files in the repo and link them into place

## Agentic engineering workflow

This repo also bootstraps the terminal-centric, multi-agent workflow described in ["L8 Principal's Agentic Engineering Workflow"](https://www.youtube.com/watch?v=iQyg-KypKAA), and is where I track changes to it over time:

- **Agent harnesses**: [Claude Code](https://claude.ai), [Codex CLI](https://github.com/openai/codex), [Pi](https://github.com/earendil-works/pi), and [OpenCode](https://github.com/sst/opencode)
- **Session management**: `tmux`, configured declaratively via `programs.tmux` in `nix/user.nix` (vi copy-mode, mouse support, session persistence via `tmux-resurrect`/`tmux-continuum`)
- **Parallel work**: [Treehouse](https://github.com/kunchenguid/treehouse) for disposable git worktrees per agent session, pulled in as a Nix flake input. `twget [label]` and `twreturn` (shell functions in `nix/user.nix`) wrap Treehouse's lease mode with tmux, so a lease opens straight into a new tmux window and returning it closes that window
- **Planning & review pipeline**: [Lavish](https://github.com/kunchenguid/lavish-axi) (interactive HTML planning artifacts) and [No Mistakes](https://github.com/kunchenguid/no-mistakes) (review/test/docs/PR pipeline)
- **Long-running agents**: [Good Night, Have Fun](https://github.com/kunchenguid/gnhf) for unattended agent loops against a stop condition
- **Agent-ergonomic tools**: the [AXI](https://github.com/kunchenguid/axi) family (`gh-axi`, `chrome-devtools-axi`) and the [Vercel `skills` CLI](https://github.com/vercel-labs/skills) for installing/managing agent skills
- **Voice input**: [OpenSuperWhisper](https://github.com/Starmel/OpenSuperWhisper), a local Whisper dictation app, installed as a Homebrew cask

`setup/mac.sh` installs the npm-distributed pieces (Codex, Pi, `skills`, `gnhf`, `no-mistakes`) and registers the AXI-family skills globally. Homebrew (`nix/host.nix`) handles OpenCode and OpenSuperWhisper. Treehouse is a proper Nix package via the flake input.

## Personal fork, not a template

The upstream repo intentionally excludes editor config, personal scripts, agent memory files, and other workflow-specific material so it stays usable as a generic starting point for other people. That constraint doesn't apply here — this fork is for my use only, so going forward I plan to fold those things in too: editor config, personal scripts, agent memory files (`~/.claude/CLAUDE.md` and friends), and other personal preferences as they stabilize enough to be worth versioning.

The one thing that stays out regardless: secrets and tokens. Nothing that grants access to an account or service belongs in this repo, personal-use or not.

## Repo structure

- `setup/mac.sh` — bootstrap a fresh Mac, including agent harnesses and workflow CLIs
- `flake.nix` — top-level Nix wiring (nixpkgs, nix-darwin, home-manager, treehouse)
- `nix/host.nix` — machine-level macOS config (nix-darwin), Homebrew brews/casks
- `nix/user.nix` — user environment: packages, shell, git, tmux, fonts, dotfiles (Home Manager)
- `files/.config/wezterm/wezterm.lua` — WezTerm config linked into place
- `files/agents/AGENTS.md` — global agent memory file, symlinked into every harness's expected location
- `files/agents/OPINIONS.md`, `files/agents/VOICE.md` — referenced conditionally from `AGENTS.md`, symlinked to `~/OPINIONS.md` and `~/VOICE.md`
- `blog.md` — local copy of the upstream author's [blog post](https://open.substack.com/pub/kunchenguid/p/how-i-built-a-reproducible-mac-setup?utm_campaign=post-expanded-share&utm_medium=web)

## Setting up a new Mac from this repo

```bash
git clone git@github.com:Seb-Yan/dotfiles-mac-nix.git ~/github/dotfiles-mac-nix
cd ~/github/dotfiles-mac-nix
bash setup/mac.sh
```

This repo targets Apple Silicon (`system = "aarch64-darwin"` in `flake.nix`).

The script will:

- install [Determinate Nix Installer](https://determinate.systems/nix-installer/) if needed
- install [Homebrew](https://brew.sh/) if needed
- apply the `nix-darwin` + Home Manager config
- install [`nvm`](https://github.com/nvm-sh/nvm) and a default Node.js version if needed

On a fresh machine, Homebrew and `npm install -g` (used for the agent harnesses below) are both being set up on this very first run, so their bin directories aren't on `PATH` yet in your current shell. Open a new terminal window/tab after the script finishes so the shell picks up the updated `PATH`.

## How I manage changes later

After the initial bootstrap, the usual workflow is:

1. edit the Nix config
2. run:

```bash
rebuild
```

This alias is included in the shell config and expands to the repo path used in this guide:

```bash
sudo /run/current-system/sw/bin/darwin-rebuild switch --flake ~/github/dotfiles-mac-nix#mac
```

## Where to add new tools

My rough rule of thumb:

- use **Home Manager / Nix** for reproducible baseline CLI tools, fonts, shell utilities, and user environment packages
- use **Homebrew** for GUI apps and macOS-native tools that fit naturally there
- use **ecosystem-specific package managers** like `npm` when that is the right abstraction for the tool

A good setup does not force every tool through one package manager. It just makes the ownership of each layer clear.

## Why this setup looks like this

I wanted a setup that was:

- reproducible on a new Mac
- structured enough to maintain
- pragmatic about macOS
- a living record of how my workflow (especially the agentic one) evolves, not a one-time snapshot

## Related

- Upstream template this was forked from: <https://github.com/kunchenguid/dotfiles-mac-nix>
- Long-form write-up of the original design: [blog post](https://open.substack.com/pub/kunchenguid/p/how-i-built-a-reproducible-mac-setup?utm_campaign=post-expanded-share&utm_medium=web)
- This fork: <https://github.com/Seb-Yan/dotfiles-mac-nix>
