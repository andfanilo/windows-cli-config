# My Modern Ubuntu CLI

## Ubuntu 24.04 LTS + Zsh + Starship + Neovim

This protocol sets up a modern, productive CLI environment on a fresh Ubuntu 24.04 LTS machine. It mirrors the Windows Terminal setup from `README.md`, adapted for Linux with Homebrew as the package manager for latest Rust CLI tools.

**Font**: Install [Cascadia Code NF](https://github.com/microsoft/cascadia-code/releases) (TTF). *(Required for Starship icons).*

---

## Phase 1: System Basics (apt)

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Install essentials
sudo apt install -y \
  zsh git curl wget unzip fontconfig build-essential \
  make gcc jq tree htop tmux xclip software-properties-common

# 3. Install Neovim (latest from PPA) + kickstart dependencies
sudo add-apt-repository ppa:neovim-ppa/unstable -y
sudo apt update
sudo apt install -y neovim ripgrep fd-find

# 4. Install VS Code
sudo snap install code --classic

# 5. Set Zsh as default shell
chsh -s $(which zsh)
```

Log out and back in for the shell change to take effect.

---

## Phase 2: Homebrew + CLI Tools

Homebrew for Linux is the easiest way to get the latest versions of modern CLI tools without hunting for PPAs.

```bash
# 1. Install Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add Homebrew to current session
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# 2. Install the "Rust" Stack
# starship: Prompt | nvim: Editor | modern replacements for ls, cat, cd, grep, find
brew install starship fzf zoxide eza

# 3. CLI Apps
brew install bat bottom delta lazygit gh tldr
```

Wire `delta` to `git`:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

git config --global core.pager delta
git config --global interactive.diffFilter "delta --color-only"
git config --global delta.navigate true
git config --global delta.side-by-side true
```

---

## Phase 3: Shell Configuration (Zsh)

**Zsh config file:** `~/.zshrc`

> **Note:** Append the following to the end of your existing `~/.zshrc` — don't replace it. Zsh's initial setup may have added `compinit` and other defaults that are fine to keep. However, **remove or comment out** any existing prompt setup (`promptinit`, `prompt adam1`, etc.) as it will override Starship.

```bash
# ── Homebrew ──
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# ── History ──
HISTSIZE=10000
SAVEHIST=10000
HISTFILE=~/.zsh_history
setopt SHARE_HISTORY
setopt HIST_IGNORE_DUPS
setopt HIST_IGNORE_SPACE

# ── Environment ──
export EDITOR="code --wait"

# ── Aliases ──
alias v="nvim"
alias c="code"
alias ls="eza --icons --group-directories-first"
alias ll="eza --icons --group-directories-first -la"
alias cat="bat --style=plain --paging=never"
alias btm="btm"
alias fd="fdfind"

# ── Zoxide (replaces cd) ──
eval "$(zoxide init zsh --cmd cd)"

# ── FZF ──
eval "$(fzf --zsh)"
export FZF_DEFAULT_COMMAND="fd --type f --strip-cwd-prefix --hidden --follow --exclude .git"
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
export FZF_CTRL_T_OPTS="--preview 'bat --color=always --style=numbers --line-range=:500 {} 2>/dev/null || echo [binary file]'"

# ── Zsh Plugins (lightweight, no framework) ──
# Auto-download if missing
ZSH_PLUGIN_DIR="$HOME/.zsh/plugins"
mkdir -p "$ZSH_PLUGIN_DIR"

if [ ! -d "$ZSH_PLUGIN_DIR/zsh-autosuggestions" ]; then
  git clone https://github.com/zsh-users/zsh-autosuggestions "$ZSH_PLUGIN_DIR/zsh-autosuggestions"
fi
if [ ! -d "$ZSH_PLUGIN_DIR/zsh-syntax-highlighting" ]; then
  git clone https://github.com/zsh-users/zsh-syntax-highlighting "$ZSH_PLUGIN_DIR/zsh-syntax-highlighting"
fi

source "$ZSH_PLUGIN_DIR/zsh-autosuggestions/zsh-autosuggestions.zsh"
source "$ZSH_PLUGIN_DIR/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh"

# ── Keybindings ──
bindkey '^[[A' history-search-backward
bindkey '^[[B' history-search-forward
bindkey '^I'   menu-complete
bindkey '^H'   backward-kill-word       # Ctrl+Backspace
bindkey '^[[1;5D' backward-word          # Ctrl+Left
bindkey '^[[1;5C' forward-word           # Ctrl+Right

# ── Prompt (MUST be last in .zshrc — move below nvm/bun if they got appended above) ──
eval "$(starship init zsh)"
```

**Starship File:** `~/.config/starship.toml`

```toml
add_newline = true

[character]
success_symbol = "[❯](bold magenta)"
error_symbol = "[❯](bold red)"

[directory]
style = "bold cyan"
```

---

## Phase 4: Runtimes & Dev Tools

### Node.js (nvm + npm)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.4/install.sh | bash

# Restart shell or source nvm
source ~/.zshrc

nvm install --lts
nvm use --lts

# tree-sitter-cli (required by Neovim kickstart, apt version is too old)
npm install -g tree-sitter-cli
```

### Bun

```bash
curl -fsSL https://bun.sh/install | bash
```

### Python (uv)

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

---

## Phase 5: Neovim

```bash
# 1. Wipe any previous config
rm -rf ~/.config/nvim ~/.local/share/nvim

# 2. Clone Kickstart
git clone https://github.com/nvim-lua/kickstart.nvim.git ~/.config/nvim

# 3. Detach from upstream
rm -rf ~/.config/nvim/.git

# 4. Launch — plugins install automatically on first run
nvim
```

---

## Phase 6: AI / Agentic Coding Tools (npm)

```bash
npm install -g @anthropic-ai/claude-code
npm install -g @google/gemini-cli

npm install -g @mariozechner/pi-coding-agent
npm install -g opencode-ai@latest 
```

---

## Phase 7: tmux

**tmux config file:** `~/.tmux.conf`

```bash
# Modern defaults
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"
set -g mouse on
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on
set -g history-limit 10000
set -s escape-time 0

# Prefix: Ctrl+a (easier than Ctrl+b)
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Intuitive splits
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Vim-style pane navigation
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Reload config
bind r source-file ~/.tmux.conf \; display "Config reloaded"
```

---

## Daily Workflow

* **Inside Neovim**: Neo-tree with `Space + e`, Telescope with `Space + sf`
* **In Terminal**: `Ctrl+T` to fuzzy-find files, `Ctrl+R` to search history
* **Navigate**: `cd <partial-name>` — zoxide learns your habits
* **Git**: `lazygit` for a full TUI, `delta` for beautiful diffs
* **System Monitor**: `btm` (Bottom) or `htop`
* **Multiplexer**: `tmux` — persist sessions across SSH disconnects
* **To Update Everything**: `brew upgrade`

