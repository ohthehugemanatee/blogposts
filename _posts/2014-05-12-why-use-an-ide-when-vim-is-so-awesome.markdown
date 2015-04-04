---
layout: post
title: "Why use an IDE when vim is so awesome?"
date: 2014-05-12 14:08:39 +0200
comments: true
categories: 
- programming
---
One thing I've never really understood is IDEs. I mean, I get it when you have a complex, emulated environment as a key part of your development. But for the majority of us who write in high level languages like PHP or Ruby, I just don't see the benefit. Why use some mouse-y, graphics dependent piece of software when you can use the ubiquitous Vim?

I use Vim because:

* It can do everything any IDE I've seen can do. Syntax highlighting, code completion, easy documentation access and writing, [debugger support](http://www.vim.org/scripts/script.php?script_id=2508)...
* It's available on every machine I work with (OSX/Linux/Unix) by default
* The configuration is portable, so I can have the same, totally customized environment on every machine I work with.
* It's just way faster and more elegant than any IDE I've seen.

I'm open to learning new things, and if I'm missing out on some huge productivity benefit by sticking with Vim, I'd like to know about it. So this post will lay out my particular Vim setup, in the hopes that someone will comment with an IDE that could do more for me.

### Configuration portability

It's worth noting that I keep all my standard configurations in a git repo. Specifically I mean my bash configuration, my ssh configuration, and my vim config. This means that when I set up a new machine, I can feel right at home with a simple "git clone". IF you don't do this yet, you're suffering needlessly.

All this to say, bear in mind the big advantage that with one command, I can have my totally customized Vim "IDE" up and running on any serious (ie non-Windows) machine.

### Global behaviors

I love the customizability of vim. Here's the first section of my ~/.vimrc file, which sets up the basic editor behaviors just the way I like 'em:

``` vim ~/.vimrc
" Editor behaviors
set nocompatible " to get all the Vim-only options
syntax enable " enable syntax highlighting
filetype plugin indent on "enable filetype detection
set showmode  " shows the current mode
set backspace=indent,eol,start "backspaces behave like backspaces
set hidden "good multi-file behaviors
set wildmenu "better command line completion
set wildmode=list:longest " completion acts like the shell
set ignorecase  "case-insensitive searching
set smartcase "unless there's a capital letter there
set ruler "show cursor position in the corner
set hlsearch "highlight search matches
set incsearch "highlight search matches as I type
set laststatus=2 " Always show a status line at the bottom
set statusline=[%n]\ %<%.99f\ %h%w%m%r%y\ %{exists('*CapsLockStatusline')?CapsLockStatusline():''}%=%-16(\ %l,%c-%v\ %)%P
set backupdir =~/.vim/backup
set wrap  "linewraps
set scrolloff=5 "always show 5 lines before/after the cursor
set title "update term title
set visualbell "turn off audio beeps

```

I also like the [solarized color scheme](http://ethanschoonover.com/solarized):

``` vim ~/.vimrc
"Solarized VIM color scheme
set background=dark
let g:solarized_termcolors=16
colorscheme solarized
```

These settings apply to everything I edit, no matter where, from Markdown to text.

### Drupal-specific behaviors

I base my Drupal config on the great [vimrc](https://drupal.org/projects/vimrc) project for Drupal. It helped me get set up with Drupal API specific features like:

* syntax highlighting
* in-editor access to API documentation
* auto-completion (Vim's normal PHP auto-complete enhanced with Drupal-specific keywords)
* integration with Drush, the Drupalist's command line tool of choice
* code snippet auto-completion. For example, I can type hook_block, press TAB, and prepopulate implementations of all the block hooks, complete with documentation (doxygen) blocks.

This is easy enough to set up for yourself with a couple of Drush commands (see the project's page on drupal.org), so I'm just going to leave the "tip of the iceberg" that actually shows in my .vimrc file.

``` vim ~/.vimrc
" Following lines added by drush vimrc-install
set nocompatible
call pathogen#infect('/$HOME/.drush/vimrc/bundle/{}')
call pathogen#infect('/$HOME/.vim/bundle/{}')
" End of vimrc-install additions.

source $VIMRUNTIME/vimrc_example.vim
filetype plugin on
filetype plugin indent on
```

I also have a couple of manually entered behaviors: double space for indents, and Drupal Coding standards in Syntastic:

``` vim ~/.vimrc
" Drupal coding standards for syntastic
let g:syntastic_phpcs_conf="--standard=DrupalCodingStandard"

"Tab sizes - use 2 spaces for tab spacing
set softtabstop=2
set tabstop=2
set shiftwidth=2
set expandtab "save as spaces rather than tabs
```

### Other IDE-like behaviors

I find I don't need much more than this setup, but it is nice to have a file explorer. There are a few plugin options for Vim, but I always liked the built in netrw. I customize it using this snippet from [detached head](http://ivanbrennan.github.io/blog/2014/01/16/rigging-vims-netrw/):

``` vim ~/.vimrc
" File explorer with \ [TAB] or \ `
fun! VexToggle(dir)
  if exists("t:vex_buf_nr")
    call VexClose()
  else
    call VexOpen(a:dir)
  endif
endf

fun! VexOpen(dir)
  let g:netrw_browse_split=4    " open files in previous window
  let vex_width = 25

  execute "Vexplore " . a:dir
  let t:vex_buf_nr = bufnr("%")
  wincmd H

  call VexSize(vex_width)
endf
noremap <Leader><Tab> :call VexToggle(getcwd())<CR>
noremap <Leader>` :call VexToggle("")<CR>

fun! VexClose()
  let cur_win_nr = winnr()
  let target_nr = ( cur_win_nr == 1 ? winnr("#") : cur_win_nr )

  1wincmd w
  close
  unlet t:vex_buf_nr

  execute (target_nr - 1) . "wincmd w"
  call NormalizeWidths()
endf

fun! VexSize(vex_width)
  execute "vertical resize" . a:vex_width
  set winfixwidth
  call NormalizeWidths()
endf

fun! NormalizeWidths()
  let eadir_pref = &eadirection
  set eadirection=hor
  set equalalways! equalalways!
  let &eadirection = eadir_pref
endf
augroup NetrwGroup
  autocmd! BufEnter * call NormalizeWidths()
augroup END

let g:netrw_liststyle=0         " thin (change to 3 for tree)
let g:netrw_banner=0            " no banner
let g:netrw_altv=1              " open files on right
let g:netrw_preview=1           " open previews vertically

" Change directory to the current buffer when opening files.
set autochdir

```

I also like to improve the auto-complete behavior with this snippet from [vim.wikia.com](http://vim.wikia.com/wiki/Omni_completion) and linked pages:

``` vim ~/.vimrc
" Enable omni completion (<C-X><C-O> when in Insert mode)
set omnifunc=syntaxcomplete#Complete
" Autocomplete behavior - complete as you type, use Enter to select.
set completeopt=longest,menuone
inoremap <expr> <CR> pumvisible() ? "\<C-y>" : "\<C-g>u\<CR>"
inoremap <expr> <C-n> pumvisible() ? '<C-n>' :
  \ '<C-n><C-r>=pumvisible() ? "\<lt>Down>" : ""<CR>'

inoremap <expr> <M-,> pumvisible() ? '<C-n>' :
  \ '<C-x><C-o><C-n><C-p><C-r>=pumvisible() ? "\<lt>Down>" : ""<CR>'
```

### IDE Alternatives?

I'm willing to look at "real" IDEs that can make my coding life easier, faster, and more efficient. But every time, I keep coming back to vim. Maybe it's because it's a part of my consistent environment everywhere. In any case, I'm interested in any ideas offered in the comments. :) So what alternatives should I check out? Or what could I do to make my Vim IDE even better?
