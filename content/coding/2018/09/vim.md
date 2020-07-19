---
title: "无处不在的 vim"
date: 2018-09-09T09:48:22+08:00
author: Alvin
keywords: vim,vim插件,编程,clion,vscode,ideavim,eclipse,vrapper,visual studio,vsvim
---

[Vim](https://www.vim.org/) 是个不错的文本编辑器，它最方便之处是，不用上下左右键就可以快速游走于文本之中。

## vim 配置

vim 的配置文件一般是 home 目录下的 .vimrc，参考配置如下：
```sh
set nocompatible
set fencs=utf-8,chinese,latin-1

"save file without permission"
cmap w!! w !sudo tee % > /dev/null <CR>

iab #i #include
iab #d #define
iab #p #pragma

inoremap jj <Esc>
inoremap <F1> <Esc>
nnoremap <F1> <Esc>
nnoremap ; :
nnoremap H ^
nnoremap L $

"share clips with windows"
set clipboard+=unnamed

"backspace and cursor keys wrap to previous/next line"
set backspace=indent,eol,start whichwrap+=<,>,[,]

set history=128
set nobackup
setlocal noswapfile
set bufhidden=hide
set confirm

set number
set smartindent
set tabstop=4
set shiftwidth=4
"set nofoldenable

set incsearch
set nohlsearch
"set ignorecase

set showcmd
set showmatch
set matchtime=1          "0.1s

set wildmenu             "auto complete for cmd
set ruler
"set cmdheight=1

set scrolloff=3
set fillchars=vert:\ ,stl:\ ,stlnc:\

if has('mouse')
  set mouse=c
endif

set selection=exclusive
set selectmode=mouse,key

set report=0
set visualbell
set noerrorbells
set shortmess=atI
set vb t_vb=

filetype on              "detect file type
filetype indent on       "indent for specific file types
syntax on
```

值得一提的是 `inoremap jj <Esc>`，在 insert 模式下，输入 `jj` 切换回 normal 模式。另外将 `F1` 也映射成了 `Esc`，因为这两个键靠得太近，经常误按 F1 跳出一个帮助页面。

## 各种 IDE 的 vim 插件

### Eclipse / Vrapper

[Eclipse](https://www.eclipse.org/) 是用 java 开发的一个开源 IDE，有一款开源的 vim 插件 [Vrapper](https://github.com/vrapper/vrapper/)，问题有点多，可能大部分是 Eclipse 本身的 Bug 导致的。

### CLion / Ideavim

[Jetbrains](https://www.jetbrains.com/) 的 CLion，PyCharm，WebStorm，GoLand 等，体验都不错。此系列 IDE 有一个专用的 vim 插件 [ideavim](https://github.com/JetBrains/ideavim)，可能是众多 vim 插件中最稳定的一个。参考配置 .ideavimrc 如下：
```sh
set nocompatible
set nohlsearch
set incsearch
set visualbell
set noerrorbells

vnoremap <Tab> ><Esc>gv
inoremap jj <Esc>
inoremap <F1> <Esc>
nnoremap <F1> <Esc>
nnoremap ; :
nnoremap H ^
nnoremap L $
```

另，CLion 依赖于 CMake，需要在代码根目录创建一个 CMakeLists.txt 文件，内容大致如下：
```sh
cmake_minimum_required(VERSION 3.8)
project(xxx)

set(CMAKE_CXX_STANDARD 11)

file(GLOB_RECURSE SOURCE_FILES
    *.h *.cc *.c *.cpp
)

include_directories(.)

add_library(xxx ${SOURCE_FILES})
```

### Visual Studio Code / VsCodeVim

[Visual Studio Code](https://code.visualstudio.com/) 写 markdown，非常流畅。

在 vscode 的 Extensions 栏搜索 `vim`，第一个出现的应该就是 VsCodeVim。经过几年的发展，目前已经比较稳定，体验比 ideavim 稍差。

可以在 vscode 的 User Settings 中配置 vim：
```sh
{
    "editor.minimap.enabled": false,
    "vim.useCtrlKeys": false,
    "vim.overrideCopy": false,
    "vim.ignorecase": false,
    "vim.insertModeKeyBindings": [
        {
            "before": ["j","j"],
            "after": ["<Esc>"]
        }
    ],
    "vim.normalModeKeyBindingsNonRecursive": [
        {
            "before": [";"],
            "after": [":"]
        },
        {
            "before": ["H"],
            "after": ["^"]
        },
        {
            "before": ["L"],
            "after": ["$"]
        }
    ],
    "workbench.colorTheme": "Monokai",
}
```
