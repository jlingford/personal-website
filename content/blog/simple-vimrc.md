+++
title = 'A basic .vimrc to get you up and running'
author = 'by James Lingford'
date = 2024-01-17T20:52:57+11:00
draft = false
toc = false
math = true
+++

To follow up my post about how to make [a simple TMUX config](https://www.jameslingford.com/blog/tmux-config/),
I thought it would be fun to follow that up with another quick post about making a simple `.vimrc` config for those completely new to Vim.

## Why Vim?

Vim comes preinstalled with most Linux systems, whereas fancier editors like Neovim are not.
This mean that if you ever do command line work in remote servers, it's likely that you won't have user permissions to install Neovim.
There really is no escaping from Vim!
Now I love Vim, but even I recognise that vanilla Vim (with no configuration) isn't pretty to look at and it lacks some important quality-of-life features.
Thankfully, configuring Vim is extremely straightforward and not as daunting as Neovim with its Lua language syntax.
All configuration can be done in the `.vimrc` file in the home directory, which makes it easy to get Vim running how you like it in any remote server you work in.

## My simple `.vimrc`

```vim
" Use space for leader key
let mapleader=" "
filetype off "required for Vundle
" General visual look of Vim
set number relativenumber
set ruler
set noerrorbells
set visualbell
set laststatus=2
set showmode
set splitbelow splitright
" Text searching options
set incsearch
set ignorecase
set smartcase
set showmatch
" Syntax and formatting
syntax on
set encoding=utf-8
set formatoptions=tcqrn1
set hidden
" Tabs and indenting
set smartindent
set tabstop=4
set shiftwidth=4
set softtabstop=4
set expandtab
set noshiftround
set scrolloff=3
" Command line completion options
set showcmd
set wildmenu
" Colors
set t_Co=256
set background=dark
let g:solarized_termcolors=256
let g:solarized_termtrans=1

" Remappings
imap JJ <ESC> "hit double upper case J in quick succession to exit insert mode.
" Autocomplete brackets and quotation marks
inoremap ( ()<ESC>hli
inoremap { {}<ESC>hli
inoremap [ []<ESC>hli
inoremap ' ''<ESC>hli
inoremap " ""<ESC>hli
inoremap ` ``<ESC>hli
" Swap colon and semicolon, makes entering command mode quicker
nnoremap ; :
nnoremap : ;
" If indenting a line while in visual mode, don't exit visual mode
vnoremap > >gv
vnoremap < <gv


" Plugins
" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" Vundle plugin to manage vundle, required
Plugin 'VundleVim/Vundle.vim'

" Add custom plugins here
Plugin 'tpope/vim-fugitive'
Plugin 'tpope/vim-surround'
Plugin 'tpope/vim-commentary'
Plugin 'morhetz/gruvbox'
Plugin 'christoomey/vim-tmux-navigator'

call vundle#end()
filetype plugin indent on "required
```

If you are unsure of what any of these settings do, such as "`set smartindent`" for example, you can simply type `:help smartindent` to find out more.

I think the plugins I've included are great and provide features I can't do without.
1. [vim-fugitive](https://github.com/tpope/vim-fugitive): provides the ability to use Git inside the Vim editor.
2. [vim-surround](https://github.com/tpope/vim-surround): allows vim-like ability to surround text with brackets, quotation marks, or other tags.
3. [vim-commentary](https://github.com/tpope/vim-commentary): allows vim-like ability to toggle on/off lines of code with comments.
4. [gruvbox](https://github.com/morhetz/gruvbox): just a classic colourscheme that looks nice ;)
5. [vim-tmux-navigator](https://github.com/christoomey/vim-tmux-navigator): makes navigating between TMUX and Vim windows with `<Ctrl-h/j/k/l>` (plugin must also be included in .tmux.conf file to work).

The fun of Vim is that you can endlessley configure and extend your setup, so [check out what other plugins you could make use of](https://github.com/akrawchyk/awesome-vim).

May you live your day with great vim.

