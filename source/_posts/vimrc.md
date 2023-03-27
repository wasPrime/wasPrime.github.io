---
title: Configurate Vim
categories:
- configuration
tags:
- vim
---

## Install `plug.vim` file

```bash
curl https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim > ~/.vim/autoload
```

## Configurate `.vimrc` file

{% note success %}
Copy the below configuration to `~/.vimrc`.
{% endnote %}

```vim
call plug#begin('~/.vim/plugged')
" 在这里使用 Plug "github用户/项目名" 的方式引入插件

" 彩虹括号
Plug 'luochen1990/rainbow'

" 历史记录
"Plug 'mhinz/vim-startify'

"One Dark Theme
Plug 'joshdick/onedark.vim'
call plug#end()


"设置配色，这里选择的是 desert，也有其他方案，在 vim 中输入 :color 再敲 tab 键可以查看
"color desert
color onedark

"设置背景色，每种配色有两种方案，一个 light、一个 dark
set background=dark

let g:rainbow_active = 1 "0 if you want to enable it later via :RainbowToggle

"传说中的去掉边框用下边这一句
set go=

"打开语法高亮
syntax on

"显示行号
set number

"设置缩进有三个取值cindent(C 风格)、smartindent(智能模式，其实不觉得有什么智能)、autoindent(简单的与上一行保持一致)
set cindent

"用空格键替换制表符
:set expandtab

"制表符占 4 个空格
set tabstop=4

"默认缩进 4 个空格大小
set shiftwidth=4

"增量式搜索
set incsearch

"高亮搜索
set hlsearch

"有时中文会显示乱码，用一下几条命令解决
let &termencoding=&encoding
set fileencodings=utf-8,gbk

"很多插件都会要求的配置检测文件类型
:filetype on
:filetype plugin on
:filetype indent on

"下边这个很有用可以根据不同的文件类型执行不同的命令
"例如：如果是 C/C++ 类型
:autocmd FileType c,cpp :set foldmethod=syntax
:autocmd FileType c,cpp :set number
:autocmd FileType c,cpp :set cindent

"例如：如果是 Python 类型
:autocmd FileType python :set number
:autocmd FileType python :set foldmethod=syntax
:autocmd FileType python :set smartindent
```

## Vim Plugin Commands

```vim
<!-- Install specific plugin such as  -->
<!-- :PlugInstall gist-vim -->
:PlugInstall <plugin_name>
<!-- Install all plugins specified in .vimrc -->
:PlugInstall

<!-- Remove plugin -->
<!-- Note that remove or comment out plugin configuration in .vimrc in advance -->
:PlugClean <plugin_name>

<!-- Udgrade vim-plug itself -->
:PlugUpgrade

<!-- Update all plugins -->
:PlugUpdate

<!-- Review the infomation of installed plugin -->
:PlugStatus <plugin_name>
```
