+++
title = 'Vim Guide'
description = 'Guide to using Vim'
+++

Vim (Vi Improved)

`Vim` stands for vi (Improved) just like its name it stands for an improved version of the `vi` text editor command.

Lightweight

Start Vim
```bash
vim
```

## Vim Search Patterns

### Moving the Cursor

h or left arrow - move left one character
k or up arrow - move up one line
j or down arrow - move down one line
l or right arrow - will move you right one character

### Different Vim Modes

I -  Enter INSERT mode from command mode
esc - Go back to command mode
v - visual mode

## Vim Appending Text

In enter while in command mode and will bring you to insert mode.

I - insert text before the cursor
O - insert text on the previous line
o - insert text on the next line
a - append text after cursor
A - append text at the end of the line

## Vim editing

x - used to cut the selected text also used for deleting characters
dd - used to delete the current line
y - yank or copy whatever is selected
yy - yank or copy the current line
p - paste the copied text before the cursor


## Vim Saving and exiting

:w - writes or saves the file
:q - quit out of vim
:wq - write and then quit
:q! - quit out of vim without saving the file
ZZ - equivalent of :wq, but one character faster

u - undo your last action
Ctrl-r - redo your last action
:% sort - Sort lines

# Vim Splits
Add to .vimrc for different key mappings for easy navigation between splits to save a keystroke. So instead of `ctrl-w` then `j`, it’s just `ctrl-j`:
```
nnoremap <C-J> <C-W><C-J> 
nnoremap <C-K> <C-W><C-K> 
nnoremap <C-L> <C-W><C-L> 
nnoremap <C-H> <C-W><C-H>
```

# Open file in new split

```
:vsp filename
```


https://github.com/preservim/nerdtree

## Find and Replace
https://linuxize.com/post/vim-find-replace/

# Find and Replace Text in File(s) with Vim

## Find and replace in a single file

Open the file in Vim, this command will replace all occurances of the word "foo" with "bar".
```
:%s/foo/bar/g
```

% - apply to whole file
s  - substitution
g - operate on all results

## Find and replace a string in all files in current directory

In vim, select all files with args. Use regex to select the files you want. Select all files with \*
```
:args *
```

You can also select all recursively:
```
:args **
```

Run :args to see which files are selected"
```
:args
```

### Perform substitution with argdo

This applies the replacement command to all selected args:
```
:argdo %s/foo/bar/g | update
```

# Nerd Tree Plugin
Add to .vimrc
```
call plug#begin()
Plug 'preservim/nerdtree'

call plug#end()

nnoremap <leader>n :NERDTreeFocus<CR>
nnoremap <C-n> :NERDTree<CR>
nnoremap <C-t> :NERDTreeToggle<CR>
nnoremap <C-f> :NERDTreeFind<CR>
```

## Vim Calendar

dhttps://blog.mague.com/?p=602

Add to vim.rc
```
:auto FileType vim/wiki map d :Vim/wikiMakeDiaryNote
function! ToggleCalendar()
  execute ":Calendar"
  if exists("g:calendar_open")
    if g:calendar_open == 1
      execute "q"
      unlet g:calendar_open
    else
      g:calendar_open = 1
    end
  else
    let g:calendar_open = 1
  end
endfunction
:auto FileType vim/wiki map c :call ToggleCalendar()i
```

## Vimwiki

## Cheat sheet
http://thedarnedestthing.com/vimwiki%20cheatsheet

### Set up

Make sure git is installed? [https://github.com/git-guides/install-git](https://github.com/git-guides/install-git "https://github.com/git-guides/install-git")

### Check git version

`git --version` Check git version

### dnf git install

sudo dnf install git-all

[https://github.com/junegunn/vim-plug](https://github.com/junegunn/vim-plug "https://github.com/junegunn/vim-plug")

Download plug.vim and put it in ~/.vim/autoload

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

### Create ~/.vimrc
`touch ~/.vimrc`

### Add to ~/.vimrc

#### Installation using [Vim-Plug](https://github.com/junegunn/vim-plug "https://github.com/junegunn/vim-plug")

Install Vim Plug
```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

Add the following to the plugin-configuration in your vimrc:
```
set nocompatible  
filetype plugin on  
syntax on

call plug#begin()  
Plug 'vimwiki/vimwiki'

call plug#end()

let mapleader=" "  
let wiki_1 = {}  
let wiki_1.path = '~/Documents/PerfectDarkMode/'  
let wiki_1.syntax = 'markdown'  
let wiki_1.ext = ''  
let wiki_2 = {}  
let wiki_2.path = '~/Documents/vim/wiki_personal/'  
let wiki_2.syntax = 'markdown'  
let wiki_2.ext = ''  
let g:vimwiki_list = [wiki_1, wiki_2]  
```

Then run `:PlugInstall`.

(leader)ws 
	select which wiki to use

## Basic Markup

```
= Header1 =
== Header2 ==
=== Header3 ===


*bold* -- bold text
_italic_ -- italic text

[wiki link](wiki%20link) -- wiki link
[description](wiki%20link) -- wiki link with description 
```

### [](https://github.com/vim/wiki/vim/wiki#lists "https://github.com/vim/wiki/vim/wiki#lists")Lists

```
* bullet list item 1
    - bullet list item 2
    - bullet list item 3
        * bullet list item 4
        * bullet list item 5
* bullet list item 6
* bullet list item 7
    - bullet list item 8
    - bullet list item 9

1. numbered list item 1
2. numbered list item 2
    a) numbered list item 3
    b) numbered list item 4 
```

For other syntax elements, see `:h vimwiki-syntax`

### Vimwiki Table of Contents

*:VimwikiTOC*
    Create or update the Table of Contents for the current wiki file.
    See |vimwiki-toc|.

Table of Contents                    *vimwiki-toc* *vimwiki-table-of-contents*

You can create a "table of contents" at the top of your wiki file.
The command |:VimwikiTOC| creates the magic header >
  = Contents =
in the current file and below it a list of all the headers in this file as
links, so you can directly jump to specific parts of the file.

For the indentation of the list, the value of |vimwiki-option-list_margin| is
used.

If you don't want the TOC to sit in the very first line, e.g. because you have
a modeline there, put the magic header in the second or third line and run
:VimwikiTOC to update the TOC.

If English is not your preferred language, set the option
|g:vimwiki_toc_header| to your favorite translation.

If you want to keep the TOC up to date automatically, use the option
|vimwiki-option-auto_toc|.

vimwiki-option-auto_toc

Key             Default value     Values~
auto_toc        0                 0, 1

Description~
Set this option to 1 to automatically update the table of contents when the
current wiki page is saved: >
  let g:vimwiki_list = [{'path': '~/my_site/', 'auto_toc': 1}]

*vimwiki-option-list_margin*

Key               Default value~
list_margin       -1 (0 for markdown)

Description~
Width of left-hand margin for lists.  When negative, the current 'shiftwidth'
is used.  This affects the appearance of the generated links (see
|:VimwikiGenerateLinks|), the Table of contents (|vimwiki-toc|) and the
behavior of the list manipulation commands |:VimwikiListChangeLvl| and the
local mappings |vimwiki_glstar|, |vimwiki_gl#| |vimwiki_gl-|, |vimwiki_gl-|,
|vimwiki_gl1|, |vimwiki_gla|, |vimwiki_glA|, |vimwiki_gli|, |vimwiki_glI| and
|vimwiki_i_<C-L>_<C-M>|.

Note: if you use Markdown or MediaWiki syntax, you probably would like to set
this option to 0, because every indented line is considered verbatim text.

*g:vimwiki_toc_header_level*

The header level of the Table of Contents (see |vimwiki-toc|). Valid values
are from 1 to 6.

The default is 1.

*g:vimwiki_toc_link_format*


The format of the links in the Table of Contents (see |vimwiki-toc|).


Value           Description~
0               Extended: The link contains the description and URL. URL
                references all levels.
1               Brief: The link contains only the URL. URL references only
                the immediate level.

Default: 0

## Key bindings

### [](https://github.com/vim/wiki/vim/wiki#normal-mode "https://github.com/vim/wiki/vim/wiki#normal-mode")Normal mode

**Note:** your terminal may prevent capturing some of the default bindings listed below. See `:h vimwiki-local-mappings` for suggestions for alternative bindings if you encounter a problem.

#### [](https://github.com/vim/wiki/vim/wiki#basic-key-bindings "https://github.com/vim/wiki/vim/wiki#basic-key-bindings")Basic key bindings

-   `<Leader>ww` -- Open default /wiki index file.
-   `<Leader>wt` -- Open default /wiki index file in a new tab.
-   `<Leader>ws` -- Select and open /wiki index file.
-   `<Leader>wd` -- Delete /wiki file you are in.
-   `<Leader>wr` -- Rename /wiki file you are in.
-   `<Enter>` -- Follow/Create /wiki link.
-   `<Shift-Enter>` -- Split and follow/create /wiki link.
-   `<Ctrl-Enter>` -- Vertical split and follow/create /wiki link.
-   `<Backspace>` -- Go back to parent(previous) /wiki link.
-   `<Tab>` -- Find next /wiki link.
-   `<Shift-Tab>` -- Find previous /wiki link.

#### [](https://github.com/vim/wiki/vim/wiki#advanced-key-bindings "https://github.com/vim/wiki/vim/wiki#advanced-key-bindings")Advanced key bindings

Refer to the complete documentation at `:h vimwiki-mappings` to see many more bindings.

## [](https://github.com/vim/wiki/vim/wiki#commands "https://github.com/vim/wiki/vim/wiki#commands")Commands

-   `:Vimwiki2HTML` -- Convert current wiki link to HTML.
-   `:VimwikiAll2HTML` -- Convert all your wiki links to HTML.
-   `:help vimwiki-commands` -- List all commands.
-   `:help vimwiki` -- General vimwiki help docs.

# Diary

### alias
alias todo='vim -c VimwikiDiaryIndex'

### Hotkeys
:VimwikiDiaryGenerateLinks
	\^w^i
	Generate links
^w^w
	open today
\^wi
	Open diary index
ctrl + up previous day
ctrl + down next day
* How to create Weekly, Monthly, and yearly notes
* How to do a template for daily
* set folder location for diary

### Diary Template
https://frostyx.cz/posts/vimwiki-diary-template


# Nested folder structure


\[dev](dev/ndex)

Say yes to make new directory

# wiki
Convert to html live and shows some design stuff
https://www.youtube.com/watch?v=A1YgbAp5YRc

https://github.com/Dynalonwiki

# Taskwarrior
https://www.youtube.com/watch?v=UuHJloiDErM
requires neovim?

## taskwiki
vimwiki integration with task warrior
https://github.com/tools-life/taskwiki
https://www.youtube.com/watch?v=UuHJloiDErM

# Ctrl P
## Install
Plug 'ctrlpvim/ctrlp.vim'
