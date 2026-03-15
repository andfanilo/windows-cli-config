# 🚀 My Modern Windows Terminal

## Windows Terminal Preview + PowerShell 7 + Starship + Neovim

This protocol transitions you from a manual, scattered setup to a modern, Scoop-managed environment. Using Scoop ensures that Neovim, Starship, and all your favorite CLI utilities are always in your `PATH` without hunting for `.exe` files.

**Font**: Install [Cascadia Code NF](https://github.com/microsoft/cascadia-code/releases) (TTF). *(Required for Starship icons).*

---

## 🧱 Phase 1: The Modern Foundation (Scoop)

Open the standard Windows PowerShell (the blue one) and run these to install your core stack.

```powershell
# 1. Install Scoop
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

# 2. Add Essential Buckets
scoop install git
scoop bucket add extras
scoop install vcredist2022

# 3. Install the "Rust" Stack
# pwsh: PowerShell 7 | starship: Prompt | nvim: Editor
scoop install pwsh starship neovim fzf ripgrep fd zoxide eza

# 4. CLI Apps
scoop install 7zip bat bottom delta ffmpeg jq less PSFzf
scoop install caddy cloudflared duckdb gh hugo mkcert terraform tldr
scoop install lazygit ttyd vhs yt-dlp

scoop bucket add stripe https://github.com/stripe/scoop-stripe-cli.git
scoop install stripe

# Terminal Preview
winget install Microsoft.WindowsTerminal.Preview

```

Wire `delta` to `git`:

```powershell
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

git config --global core.pager delta
git config --global interactive.diffFilter "delta --color-only"
git config --global delta.navigate true
git config --global delta.side-by-side true

```

---

## 🐚 Phase 2: Shell Configuration (PowerShell 7)

We use **PowerShell 7 (pwsh)** instead of Git Bash because it handles Windows native paths and long file names significantly better for modern CLI tools.

**Powershell 7 File:** `$HOME\Documents\PowerShell\Microsoft.PowerShell_profile.ps1`
*(Create this file if it doesn't exist by typing `notepad $PROFILE` in your terminal)*

```powershell
$env:EDITOR = "code --wait" 

# --- Prompt ---
Invoke-Expression (&starship init powershell)

# --- PSReadLine ---
Set-PSReadLineOption -PredictionSource HistoryAndPlugin
Set-PSReadLineOption -PredictionViewStyle InlineView
Set-PSReadLineOption -EditMode Windows
Set-PSReadLineOption -BellStyle None
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward

# --- Aliases ---
Set-Alias v nvim
Set-Alias c code

Remove-Item Alias:ls  -ErrorAction SilentlyContinue
Remove-Item Alias:cat -ErrorAction SilentlyContinue
function ls  { eza --icons --group-directories-first @args }
function ll  { eza --icons --group-directories-first -la @args }
function cat { bat --style=plain --paging=never @args 2>$null }

# --- Zoxide (replaces cd) ---
Invoke-Expression (& { (zoxide init powershell --cmd cd | Out-String) })

# --- FZF ---
$env:FZF_DEFAULT_COMMAND = "fd --type f --strip-cwd-prefix --hidden --follow --exclude .git"
$env:FZF_CTRL_T_COMMAND  = $env:FZF_DEFAULT_COMMAND
$env:FZF_CTRL_T_OPTS = "--preview 'bat --color=always --style=numbers --line-range=:500 {} 2>nul || echo [binary file]'"
Import-Module PSFzf
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+t' -PSReadlineChordReverseHistory 'Ctrl+r'

# --- Cleanup ---
$env:POWERSHELL_UPDATECHECK = 'Off'

```

**Starship File:** `$HOME\.config\starship.toml`
*(Create the folder and file if they don't exist: `mkdir -Force ~\.config; notepad ~\.config\starship.toml`)*

**Minimal config:**

```toml
add_newline = true

[character]
success_symbol = "[❯](bold magenta)"
error_symbol = "[❯](bold red)"

[directory]
style = "bold cyan"

```

**Ultra-performance config** — use after profiling with `starship timings` or `starship explain`:

```toml
# 1. Performance Tuning
command_timeout = 500
add_newline = false

# 2. Ultra-Fast Directory
[directory]
style = "bold cyan"
truncation_length = 1
truncate_to_repo = false    # Prevent walking up tree to find .git
use_logical_path = true     # Faster path resolution on Windows
read_only = ""              # Disable RO check to save disk I/O

# 3. The "Silent Git" Strategy 
# We disable the heavy scanners that cause 600ms+ lag on Windows
[git_branch]
disabled = true
[git_status]
disabled = true
[git_commit]
disabled = true
[git_state]
disabled = true

# 4. Disable Language/System Scanners (Saves ~100ms total)
[python]
disabled = true
[nodejs]
disabled = true
[rust]
disabled = true
[golang]
disabled = true
[package]
disabled = true
[docker_context]
disabled = true
[battery]
disabled = true
[username]
disabled = true
[hostname]
disabled = true

# 5. UI Elements
[character]
success_symbol = "[❯](bold magenta)"
error_symbol = "[❯](bold red)"

```

**VS Code:** Change the Integrated Default Profile to Scoop's PowerShell (`pwsh`) in **Settings > Terminal > Default Profile: Windows**.

---

## 🖥️ Phase 3: Host — Windows Terminal Preview

Windows Terminal Preview is installed via `winget` in Phase 1. This section configures it for a clean, high-performance look (the "Ghostty" aesthetic).

### GUI Settings (`Ctrl + ,`)

1. **Startup > Default profile**: Select **PowerShell** (the Scoop-installed one, *not* "Windows PowerShell 5.1").
2. **Profile: PowerShell (Scoop)**:
   * **Command line**: `pwsh.exe -NoLogo`
   * **Font face**: `Cascadia Code NF` (or `Cascadia Code NF Mono`)
   * **Font size**: `12`
   * **Cursor shape**: `Filled box`
3. **Appearance** (under the profile, untested):
   * **Background opacity**: `85%`
   * **Enable Acrylic material**: `On`

### JSON Overrides (the "Ghostty" Look, untested)

Click **Open JSON file** (bottom-left of Settings) and merge these keys into your PowerShell profile object inside the `profiles.list` array:

```json
{
    "padding": "15, 15, 15, 15",
    "scrollbarState": "hidden",
    "useAtlasEngine": true,
    "bellStyle": "none",
    "antialiasingMode": "cleartype",
    "adjustIndistinguishableColors": "indexed"
}
```

> **Tip:** Set `"defaultProfile"` at the top level to your PowerShell profile's `guid` so it launches by default.

---

## 📝 Phase 4: Neovim

Neovim was installed in Phase 1 via Scoop. This section sets up [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim) — a single-file config that bootstraps LSP, Treesitter, Telescope, and Autocomplete.

### Install Dependencies & Bootstrap

```powershell
# 1. Install build tools (Treesitter needs a C compiler)
scoop install tree-sitter mingw main/gcc

# 2. Wipe any previous config (safe on a fresh machine)
Remove-Item -Recurse -Force $env:LOCALAPPDATA\nvim      -ErrorAction SilentlyContinue
Remove-Item -Recurse -Force $env:LOCALAPPDATA\nvim-data  -ErrorAction SilentlyContinue

# 3. Clone Kickstart into the Neovim config folder
git clone https://github.com/nvim-lua/kickstart.nvim.git $env:LOCALAPPDATA\nvim

# 4. Detach from upstream so you own the config
Remove-Item -Recurse -Force $env:LOCALAPPDATA\nvim\.git

# 5. Launch Neovim — plugins install automatically on first run
nvim
```

### Troubleshooting: Treesitter GCC Errors

If Treesitter parsers fail to compile, force the compiler explicitly:

1. Edit `$env:LOCALAPPDATA\nvim\init.lua` and add the `compilers` line inside the Treesitter config:

   ```lua
   config = function()
         require('nvim-treesitter.install').compilers = { "gcc" } -- Add this
         local parsers = { 'bash', 'c', 'diff', 'html', 'lua', 'luadoc',
                           'markdown', 'markdown_inline', 'query', 'vim', 'vimdoc' }
         -- ... rest of the config
   ```

2. Set the environment variable and relaunch:

   ```powershell
   $env:CC = "gcc"
   nvim
   ```

---

## 📌 Phase 5: Daily Workflow

* **Inside Neovim**: Use Neo-tree (already in Kickstart) with Space + e.
* **In Terminal**: Use fzf. Press Ctrl + T to find any file and hit Enter to open it.
* **Large Projects**: Use `code .` or `c .` alias to open the current folder in VS Code .
* **To Edit Code**: Type `v` (alias for nvim).
* **To Search/Jump**: Use `cd [folder name]` — zoxide is wired in automatically
* **System Monitor**: Type `btm` to launch Bottom — a Rust-based task manager in your terminal. Replaces `Ctrl+Shift+Esc`.
* **To Update Everything**: Just run `scoop update *`. No installers, no websites.

# 📦 Globally installed CLI Apps

## Powershell

```powershell
irm https://astral.sh/uv/install.ps1 | iex
irm bun.sh/install.ps1 | iex

irm https://claude.ai/install.ps1 | iex
irm https://ampcode.com/install.ps1 | iex
```

## NodeJS / Bun

```powershell
npm install -g @google/gemini-cli
```

## UV

```powershell
```