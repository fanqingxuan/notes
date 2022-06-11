### 其它
积累了一个基于Plug的vim配置，分享个大家，包括但不限于如下插件

- 支持自定义主题，包括onedark、molokai、solarized等
- 自动完成匹配引号、括号
- 函数搜索
- 目录结构
- 代码美化、代码格式化
- 语法检查
- 彩虹括号

```shell
"去除vi一致性
set nocompatible
"设置 backspace可以删除任意字符
set backspace=2
syntax enable
"缩进
set autoindent
set shiftwidth=4
set tabstop=4
set expandtab
set smarttab
"设置编码
set fencs=utf-8,ucs-bom,shift-jis,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
set fileencodings=ucs-bom,utf-8,cp936
set fileencoding=utf-8
"显示行号
set number
"显示光标当前位置
set ruler
"突出当前行
set cursorline
"共享剪切板
set clipboard+=unnamed
"从不备份
set nobackup
"高亮搜索项
set hlsearch
"启用文件类型检测
filetype plugin indent on
"自动跳到上次打开的光标位置
if has("autocmd")
  au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif
endif

call plug#begin('~/.vim/plugged')
	"成对删除或插入括号,引号
	Plug 'jiangmiao/auto-pairs'
	"ctrl+p 文件搜索
	Plug 'ctrlpvim/ctrlp.vim'
	"函数搜索
	Plug 'tacahiroy/ctrlp-funky'
	"树形目录
	Plug 'preservim/nerdtree'
	Plug 'jistr/vim-nerdtree-tabs'
	"代码注释
	Plug 'tpope/vim-commentary'
	"成对符号编辑
	Plug 'tpope/vim-repeat'
	Plug 'tpope/vim-surround'
	"彩虹括号
	Plug 'luochen1990/rainbow'
	"语法检查
	Plug 'tpope/vim-pathogen'
	Plug 'scrooloose/syntastic'
	"各种主题
    Plug 'joshdick/onedark.vim'
    Plug 'altercation/vim-colors-solarized'
    Plug 'sickill/vim-monokai'
    Plug 'tomasr/molokai'
    Plug 'liuchengxu/space-vim-dark'
    Plug 'morhetz/gruvbox'
    Plug 'jpo/vim-railscasts-theme'
    Plug 'cormacrelf/vim-colors-github'
    Plug 'nanotech/jellybeans.vim'
    Plug 'rakr/vim-one'
    Plug 'jdkanani/vim-material-theme'
    Plug 'dracula/vim'
    Plug 'NLKNguyen/papercolor-theme'
	"对齐
	Plug 'godlygeek/tabular'
    "状态栏
    Plug 'vim-airline/vim-airline'
    "缩进
    Plug 'Yggdroot/indentLine'
    "多光标操作
    Plug 'terryma/vim-multiple-cursors'

    "行尾空格标红，命令模式:FixWhitespace去文件内所有行尾空格
    Plug 'bronson/vim-trailing-whitespace'
    "css color
    Plug 'ap/vim-css-color'
    "tagbar 依赖Exuberant Ctags >= 5.5
    Plug 'majutsushi/tagbar'

    "php代码格式化
    "依赖php-cs-fixer， php-cs-fixer要求php>=5.6
    "a.下载: wget https://cs.symfony.com/download/php-cs-fixer-v2.phar -O php-cs-fixer
    "b.修改mod: sudo chmod a+x php-cs-fixer
    "c.移动到PATH: sudo mv php-cs-fixer /usr/local/bin/php-cs-fixer
    Plug 'stephpy/vim-php-cs-fixer'

    "editorconfig插件
    Plug 'editorconfig/editorconfig-vim'
call plug#end()

"=========================设置主题的几种方式==============================
"solarized主题
"if has('gui_running')
"    set background=light
"else
"    set background=dark
"endif
"colorscheme solarized

"onedark包中的主题
let g:onedark_termcolors=256
colorscheme onedark

"monokai包主题
"colorscheme monokai

"molokai主题
"set t_Co=256
"colorscheme molokai

"space vim dark主题
"set t_Co=256
"colorscheme space-vim-dark

"gruvbox主题
"set t_Co=256
"set background=dark
"set background=light
"colorscheme gruvbox

"railscast主题
"set t_Co=256
"colorscheme railscasts

"github主题
"set t_Co=256
"colorscheme github

"jellybeans主题
"set t_Co=256
"colorscheme jellybeans

"one主题
"set t_Co=256
"set background=dark
"set background=light
"colorscheme one

"jdkanani/vim-material-theme
"set t_Co=256
"set background=dark
"colorscheme material-theme

"dracula主题
"set t_Co=256
"colorscheme dracula

"PaperColor主题
"set t_Co=256
"set background=light
"set background=dark
"colorscheme PaperColor

"=========================设置主题end=====================================


"树形目录配置
map <C-n> :NERDTreeToggle<CR>
" open NERDTree automatically when vim starts up if no files were specified
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
" close vim if the only window left is a NERDTree
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") && b:NERDTreeType == "primary") | q | endif
let NERDTreeHighlightCursorline=1
let NERDTreeIgnore=[ '\.pyc$', '\.pyo$', '\.obj$', '\.o$', '\.so$', '\.egg$', '^\.git$', '^\.svn$', '^\.hg$' ]
let g:NERDTreeWinPos = "right"
map <Leader>nn :NERDTreeTabsToggle<CR>
map <leader>nb :NERDTreeFromBookmark
map <leader>nf :NERDTreeFind<cr>
let g:NERDTreeMapOpenSplit = 's'
let g:NERDTreeMapOpenVSplit = 'v'
map <Leader>n <plug>NERDTreeTabsToggle<CR>
let g:nerdtree_tabs_synchronize_view=1
let g:nerdtree_tabs_synchronize_focus=0
" 是否显示隐藏文件
let NERDTreeShowHidden=1
"设置宽度
 let NERDTreeWinSize=25
"忽略一下文件的显示
let NERDTreeIgnore=['\.pyc','\~$','\.swp']

"代码注释配置
"为python和shell等添加注释
autocmd FileType python,shell,coffee set commentstring=#\ %s
"修改注释风格
autocmd FileType java,c,cpp,php set commentstring=//\ %s

"ctrl+p 文件搜索
"set wildignore+=*/tmp/*,*.so,*.swp,*.zip     " MacOSX/Linux"
let g:ctrlp_custom_ignore = {
    \ 'dir':  '\v[\/]\.(git|hg|svn|rvm)$',
    \ 'file': '\v\.(exe|so|dll|zip|tar|tar.gz)$',
    \ }
"\ 'link': 'SOME_BAD_SYMBOLIC_LINKS',
let g:ctrlp_working_path_mode=0
let g:ctrlp_match_window_bottom=1
let g:ctrlp_max_height=15
let g:ctrlp_match_window_reversed=0
let g:ctrlp_mruf_max=500
let g:ctrlp_follow_symlinks=1

"函数搜索
nnoremap <Leader>fu :CtrlPFunky<Cr>
" narrow the list down with a word under cursor
nnoremap <Leader>fU :execute 'CtrlPFunky ' . expand('<cword>')<Cr>
let g:ctrlp_funky_syntax_highlight = 1
let g:ctrlp_extensions = ['funky']

"状态栏设置
let g:airline#extensions#tabline#enabled = 1

"彩虹括号
let g:rainbow_active = 1

"语法检查配置
execute pathogen#infect()
set statusline+=%#warningmsg#
set statusline+=%{SyntasticStatuslineFlag()}
set statusline+=%*
let g:syntastic_always_populate_loc_list = 1
let g:syntastic_auto_loc_list = 1
let g:syntastic_check_on_open = 1
let g:syntastic_check_on_wq = 0

"缩进配置
let g:indentLine_color_term = 239
"let g:indentLine_char = '┆'

"多光标控制
"使用方法:ctrl+j选择下一行，ctrl+k选择前一行，ctrl+x跳过当前行，再命令行模式c进行删除替换
let g:multi_cursor_use_default_mapping=0
" Default mapping
let g:multi_cursor_next_key='<C-j>'
let g:multi_cursor_prev_key='<C-k>'
let g:multi_cursor_skip_key='<C-x>'
let g:multi_cursor_quit_key='<Esc>'

"tagbar
"F9触发，设置宽度为30
let g:tagbar_width = 30
nmap <F9> :TagbarToggle<CR>
"开启自动预览(随着光标在标签上的移动，顶部会出现一个实时的预览窗口)
let g:tagbar_autopreview = 1
"关闭排序,即按标签本身在文件中的位置排序
let g:tagbar_sort = 0
"设置tagbar的窗口显示的位置,为右边
let g:tagbar_right_=1
"打开文件自动 打开
"autocmd BufReadPost *.* call tagbar#autoopen()"

"php代码格式化配置
" If php-cs-fixer is in $PATH, you don't need to define line below
" let g:php_cs_fixer_path = "~/php-cs-fixer.phar" " define the path to the php-cs-fixer.phar

" If you use php-cs-fixer version 1.x
let g:php_cs_fixer_level = "symfony"                   " options: --level (default:symfony)
let g:php_cs_fixer_config = "default"                  " options: --config
" If you want to define specific fixers:
"let g:php_cs_fixer_fixers_list = "linefeed,short_tag" " options: --fixers
"let g:php_cs_fixer_config_file = '.php_cs'            " options: --config-file
" End of php-cs-fixer version 1 config params

" If you use php-cs-fixer version 2.x
let g:php_cs_fixer_rules = "@PSR2"          " options: --rules (default:@PSR2)
"let g:php_cs_fixer_cache = ".php_cs.cache" " options: --cache-file
"let g:php_cs_fixer_config_file = '.php_cs' " options: --config
" End of php-cs-fixer version 2 config params

let g:php_cs_fixer_php_path = "php"               " Path to PHP
let g:php_cs_fixer_enable_default_mapping = 1     " Enable the mapping by default (<leader>pcd)
let g:php_cs_fixer_dry_run = 0                    " Call command with dry-run option
let g:php_cs_fixer_verbose = 0                    " Return the output of command if 1, else an inline information.

autocmd BufWritePost *.php silent! call PhpCsFixerFixFile()
```