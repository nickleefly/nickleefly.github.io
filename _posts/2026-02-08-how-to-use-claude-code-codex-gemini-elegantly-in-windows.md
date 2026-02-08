---
layout: post
title: "How to use Claude code, codex, Gemini elegantly in windows "
date: 2026-02-08 16:20
comments: true
categories: claude, ai
---

This guide explains how to install Windows Subsystem for Linux (WSL), configure the Alacritty terminal with Tmux, set up the Berkeley Mono font, and install CLI tools for Claude, Gemini, and Codex.

# Part 1: System & Terminal Setup

## 1. Install WSL (Windows Subsystem for Linux)

1. Open PowerShell as Administrator.
2. Run the command:
   ```powershell
   wsl --install
   ```
3. Restart the computer when prompted.
4. After rebooting, a terminal window will open to set up your Ubuntu username and password.

## 2. Install Alacritty (Windows)

Alacritty is a terminal emulator that runs on Windows, connecting to WSL.

1. **Install**: Use a package manager like Winget (pre-installed on Windows 11/10) or download from GitHub.
   ```powershell
   winget install Alacritty.Alacritty
   ```

2. **Configure for WSL**:
   Find or create the config file at `%APPDATA%\alacritty\alacritty.toml` (or `.yml` for older versions).

   Add the following to launch WSL by default:
   ```toml
   [terminal]
   shell = { program = "wsl.exe", args = ["~"] }
   ```

## 3. Install Tmux (Inside WSL)

1. Open your newly installed Alacritty (it should launch into Ubuntu/WSL).
2. Update repositories and install Tmux:
   ```bash
   sudo apt update && sudo apt install tmux -y
   ```
3. create `~/.tmux.conf` file:
   ```bash
   touch ~/.tmux.conf
   ```
   and add the following content:
   ```bash
    # ==========================================
    # 1. General Settings
    # ==========================================
    set -g default-terminal "tmux-256color"
    set -ga terminal-overrides ",xterm-256color:Tc"

    # Remap prefix to Control + s
    set -g prefix C-s
    unbind C-b
    bind C-s send-prefix

    set -g focus-events on
    set -sg escape-time 10  # Increased to 10ms for better stability than 0
    set -sg repeat-time 600
    set -g history-limit 99999

    # Mouse support
    set -g mouse on
    set-option -s set-clipboard off
    unbind -T copy-mode-vi MouseDragEnd1Pane # Don't exit copy mode when dragging

    # Indexing
    set -g base-index 1
    setw -g pane-base-index 1
    set -g renumber-windows on # Automatically renumber windows when one is closed

    # ==========================================
    # 2. Key Bindings
    # ==========================================

    # Reload config
    unbind r
    bind r source-file ~/.tmux.conf \; display "Reloaded ~/.tmux.conf"

    # Splitting panes (retains your preference)
    unbind %
    bind | split-window -h -c "#{pane_current_path}"
    bind h split-window -h -c "#{pane_current_path}"
    unbind '"'
    bind - split-window -v -c "#{pane_current_path}"
    bind v split-window -v -c "#{pane_current_path}"

    # Pane resizing
    bind -r H resize-pane -L 5
    bind -r J resize-pane -D 5
    bind -r K resize-pane -U 5
    bind -r L resize-pane -R 5

    # Window Navigation (Alt + Number)
    bind -n M-1 select-window -t 1
    bind -n M-2 select-window -t 2
    bind -n M-3 select-window -t 3
    bind -n M-4 select-window -t 4
    bind -n M-5 select-window -t 5
    bind -n M-6 select-window -t 6
    bind -n M-7 select-window -t 7
    bind -n M-8 select-window -t 8
    bind -n M-9 select-window -t 9

    # Search tmux session with fzf
    bind C-j split-window -v "tmux list-sessions | sed -E 's/:.*$//' | grep -v \"^$(tmux display-message -p '#S')\$\" | fzf --reverse | xargs tmux switch-client -t"

    # Restoring Clear Screen (C-l) since vim-tmux-navigator overrides it
    bind C-l send-keys 'C-l'

    # ==========================================
    # 3. Copy Mode (Vi Style)
    # ==========================================
    setw -g mode-keys vi
    set -g status-keys vi

    unbind [
    bind Escape copy-mode
    unbind p
    bind p paste-buffer

    bind-key -T copy-mode-vi v send -X begin-selection
    # Native macOS clipboard support (no reattach-to-user-namespace needed)
    bind-key -T copy-mode-vi y send -X copy-pipe-and-cancel "pbcopy"
    bind-key -T copy-mode-vi Enter send -X copy-pipe-and-cancel "pbcopy"

    # ==========================================
    # 4. Plugins (TPM)
    # ==========================================
    set -g @plugin 'tmux-plugins/tpm'
    set -g @plugin 'christoomey/vim-tmux-navigator'
    set -g @plugin 'tmux-plugins/tmux-resurrect'
    set -g @plugin 'tmux-plugins/tmux-continuum'
    set -g @plugin 'dracula/tmux'

    # --- Plugin Settings ---
    set -g @resurrect-capture-pane-contents 'on'
    set -g @continuum-restore 'on'

    # Dracula Theme Settings (Overrides manual status bar code)
    set -g status-position top
    set -g @dracula-show-powerline true
    set -g @dracula-fixed-location "Shanghai"
    set -g @dracula-plugins "weather"
    set -g @dracula-show-fahrenheit false
    set -g @dracula-show-flags true
    set -g @dracula-show-left-icon session

    # ==========================================
    # 5. Initialization (MUST BE LAST)
    # ==========================================
    run '~/.tmux/plugins/tpm/tpm'
   ```

## 4. Install Berkeley Mono Font

> **Note**: Berkeley Mono is a paid typeface and cannot be downloaded automatically.

1. **Purchase/Download**: Get the font files from the official Berkeley Graphics site.
2. **Install in Windows**: Double-click the `.ttf` files and click **Install**.
   > **Important**: Alacritty runs on Windows, so the font must be installed in Windows, not just WSL.

3. **Update Alacritty Config**:
   Edit `%APPDATA%\alacritty\alacritty.toml`:
   ```toml
   [font]
   normal = { family = "Berkeley Mono", style = "Regular" }
   size = 12.0
   ```

# Part 2: Install AI CLI Tools

These tools require Node.js. Install a recent version in WSL using `nvm` (recommended over `apt` to avoid permission issues).

## Prerequisite: Install Node.js

```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# Install Node.js (LTS)
nvm install --lts
```

## 1. Install Claude Code (Anthropic)

Claude Code provides a CLI that is "tmux-aware".

1. **Install**:
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```

2. **Authenticate**:
   Run `claude` in the terminal. It will provide a link to authenticate via the browser.

3. **Tmux Features**: Claude Code can read other tmux panes. You can reference them in prompts like "fix the error in tmux pane 1".

## 2. Install Gemini CLI (Google)

1. **Install**:
   ```bash
   npm install -g @google/gemini-cli
   ```

2. **Authenticate**:
   Run `gemini`. It will prompt you to log in with your Google account.

## 3. Install Codex CLI (OpenAI)

> **Note**: "Codex" often refers to the model powering Copilot, but OpenAI maintains a CLI tool.

1. **Install**:
   ```bash
   npm install -g @openai/codex
   ```

2. **Setup**:
   You will need an OpenAI API key. Set it in your environment or when prompted.
   ```bash
   export OPENAI_API_KEY="your-api-key-here"
   ```

# Part 3: Usage Guide

1. **Launch**: Open Alacritty. It enters WSL automatically.
2. **Start Tmux**:
   ```bash
   tmux new -s ai-dev
   ```

## Workflow

- **Split Panes**: Press `Ctrl+s` then `h` (vertical split) or `v` (horizontal split).

## Run Tools

- **Pane 1**: Run your code/server.
- **Pane 2**: Run `claude` or `gemini`.

**Context Awareness**: When using `claude` inside tmux, it can inspect errors or logs visible in your other panes, making debugging faster.
