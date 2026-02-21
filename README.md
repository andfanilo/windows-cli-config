# 👻 Ghostty-Style Terminal Setup for Windows

**WezTerm / Windows Terminal + Git Bash + Starship**

This setup provides a GPU-accelerated, minimalist, Unix-like environment on Windows with a modern Lua-based configuration.

## 📦 Phase 1: Binaries & Assets

1. **WezTerm**: Download `wezterm-gui-*.zip` (Portable), unzip to `C:/executables`.
2. **Starship**: Download `starship-x86_64-pc-windows-msvc.zip`, move `starship.exe` to `C:/executables`.
3. **Git for Windows**: Install to default directory (`C:/Program Files/Git`).
4. **Font**: Install [Cascadia Code NF](https://github.com/microsoft/cascadia-code/releases). *(Required for Starship icons).*

---

## 🐚 Phase 2: Shell Configuration (Unified)

This configuration lives in your User home and is shared by **both** WezTerm and Windows Terminal.

**File:** `C:/Users/<User>/.bashrc`

```bash
# Add custom executables (Starship/WezTerm) to PATH
export PATH="$PATH:/c/executables"

# Initialize Starship Prompt
eval "$(starship init bash)"

```

**File:** `C:/Users/<User>/.bash_profile`

```bash
# Ensure .bashrc loads in all login shells (critical for Starship to appear)
[[ -f ~/.bashrc ]] && . ~/.bashrc

```

**File:** `C:/Users/<User>/.config/starship.toml`

```toml
add_newline = false

[character]
success_symbol = "[❯](bold magenta)"
error_symbol = "[❯](bold red)"

[directory]
style = "bold cyan"

```

This prevents the terminal from flashing or beeping when you backspace on an empty line or encounter a shell error.

**File:** C:/Users/<User>/.inputrc

```bash
# Disable the visual/audible bell in Git Bash (Readline)
set bell-style none

```

---

## 🖥️ Phase 3: Host 1 - WezTerm (The Ghostty Alternative)

**File:** `C:/Users/<User>/.wezterm.lua`

```lua
local wezterm = require 'wezterm'
local config = wezterm.config_builder()

-- Default Workspace
config.default_cwd = "C:/workspace"

-- 1. SET THE SHELL (Git Bash)
-- The --login flag ensures .bash_profile is read so Starship starts
config.default_prog = { 'C:\\Program Files\\Git\\bin\\bash.exe', '--login', '-i' }

-- 2. VISUALS (The Ghostty Look)
config.color_scheme = 'Tokyo Night'
config.font = wezterm.font('Cascadia Code NF') -- Requires a Nerd Font for icons
config.font_size = 11.0
config.line_height = 1.1

-- Make it look sleek (Transparency & Blur)
config.window_background_opacity = 0.90
config.win32_system_backdrop = 'Acrylic' -- Windows-specific blur effect
config.window_decorations = "RESIZE"     -- Removes the thick title bar
config.window_padding = {
  left = 15,
  right = 15,
  top = 15,
  bottom = 15,
}

-- 3. MOUSE & UTILITY
config.adjust_window_size_when_changing_font_size = false

config.mouse_bindings = {
  -- CTRL + Scroll Up to make font BIGGER
  {
    event = { Down = { streak = 1, button = { WheelUp = 1 } } },
    mods = 'CTRL',
    action = wezterm.action.IncreaseFontSize,
  },
  -- CTRL + Scroll Down to make font SMALLER
  {
    event = { Down = { streak = 1, button = { WheelDown = 1 } } },
    mods = 'CTRL',
    action = wezterm.action.DecreaseFontSize,
  },
}

-- Reset font size with CTRL + 0
config.keys = {
  { key = '0', mods = 'CTRL', action = wezterm.action.ResetFontSize },
}

return config

```

---

## 🏁 Phase 4: Host 2 - Windows Terminal (The Native Host)

Use this to make your new Git Bash + Starship setup the system default.

1. Open **Windows Terminal** > **Settings** (`Ctrl + ,`).
2. **Create Profile**:
* Click **Add a new profile** > **New empty profile**.
* **Name**: `Git Bash`
* **Command line**: `%PROGRAMFILES%\Git\bin\bash.exe --login -i`
* **Starting directory**: `C:\workspace`
* **Icon**: `%PROGRAMFILES%\Git\mingw64\share\git\git-for-windows.ico`


3. **Set Appearance**:
* Under **Appearance**, set **Font face** to `Cascadia Code NF`.
* Set **Opacity** to `90%` and enable **Acrylic material**.


4. **Set as Default**:
* Go to the **Startup** tab on the left sidebar.
* Set **Default profile** to `Git Bash`.
* Set **Default terminal application** to `Windows Terminal`.



---

## 📌 Phase 5: Shortcuts

* **For WezTerm**: Create a shortcut to `C:/executables/wezterm-gui.exe`. In properties, set **Start in** to `C:\workspace`. Pin to Taskbar.
* **For Windows Terminal**: Simply pin the Windows Terminal app to the taskbar. Since you set Git Bash as the default, it will launch your Starship environment automatically.
