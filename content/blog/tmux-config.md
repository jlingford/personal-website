+++
title = 'A simple TMUX configuration'
author = 'by James Lingford'
date = 2024-01-02T14:44:05+11:00
draft = false
toc = true
math = true
+++

## Why use tmux?

Tmux is a terminal multiplexer that is extremely useful when doing any remote SSH work.
For instance, what if you want to run an interactive job on a remote HPC, but don't want to lose all your work if you accidentally lose internet connection?
Tmux is the key. Run a new terminal session, do some work, detach from that session and reattach to pick up where you last left off.

The downside to tmux is that it isn't the most user-friendly of terminal applications (even by terminal application standards, in my opinion).
The default Tmux configuration is missing some key quality-of-life features like mouse scrolling, vim-like keybindings, and easy window navigation between a pane running vim and a pane running the command-line.

## My simple `.tmux.conf` configuration

Below is my quick and simple tmux configuration.
It includes some basic conveniences and some highly useful plugins without going completely overboard.
May this serve as a good starting point for your own `tmux.conf` :)

Note: first create the tmux config file with `touch ~/.tmux.conf` if you haven't done so already.
```bash
# Allow mouse scrolling
set -g mouse on

# Allow default copy and pasting behaviour
set -s set-clipboard external

# History size
set -g history-limit 10000

# Allow config relaoding with <C-b> r
bind r source-file ~/.tmux.conf

# set vi-mode
set-window-option -g mode-keys vi

# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'christoomey/vim-tmux-navigator'
set -g @plugin 'tmux-plugins/tmux-yank'
set -g @plugin 'tmux-plugins/tmux-prefix-highlight'

# Prefix highlight on status bar
set -g status-right '#{prefix_highlight} | %a %Y-%m-%d %H:%M'

# Initialize TPM
# always keep this line at the bottom of the config file
run '~/.tmux/plugins/tpm/tpm'
```
The [Tmux Plugins Manager](https://github.com/tmux-plugins/tpm) (tpm) requires some minimal setup by first running `git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm`.
After some plugins have been added to the tmux config file as seen above, the new tmux settings should take effect immediately with starting a new session. 
You can find this in my [GitHub dotfiles repo](https://github.com/jlingford/.dotfiles) too.

## Useful resources

- The [`man tmux` page](https://man7.org/linux/man-pages/man1/tmux.1.html)
- Tmux [cheatsheet](https://quickref.me/tmux.html) for common tmux commands
- The Tmux Plugins Manager repo: [tmux-plugins/tpm](https://github.com/tmux-plugins/tpm)
- "Tmux sensible" plugin: [tmux-plugins/tmux-sensible](https://github.com/tmux-plugins/tmux-sensible)
- Vim-Tmux navigator plugin for seamless pane switching: [christoomey/vim-tmux-navigator](https://github.com/christoomey/vim-tmux-navigator)
    - Note: this plugin requires a sister plugin to be added to your vim/neovim config as well! 
- Tmux yank plugin to allow copying text to system keyboard: [tmux-plugins/tmux-yank](https://github.com/tmux-plugins/tmux-yank)
- Tmux prefix highlight plugin. I find I need a visual cue when I hit the tmux prefix key (i.e. CTRL-b) otherwise I get paranoid that I didn't hit the keys right. This plugin solves that issue: [tmux-plugins/tmux-prefix-highlight](https://github.com/tmux-plugins/tmux-prefix-highlight)
