# Vim

## 插件清单

* nerdtree。文件目录树形插件
* nerdtree-git-plugin。nerdtree git插件
* vim-go。golang插件
* tagbar。标签插件，显示文件中的缩略信息
* molokai。molokai颜色主题插件

## 快速配置

{% embed url="https://github.com/wujie1993/dev-env/tree/master/vim" %}

下载配置脚本

```bash
git clone git@github.com:wujie1993/dev-env.git $GOPATH/src/github.com/wujie1993/dev-env
```

执行配置脚本

```bash
cd $GOPATH/src/github.com/wujie1993/dev-env/vim && sh ./setup.sh
```

打开vim编辑器，安装vim插件

```text
:PluginInstall
```

安装golang插件

```text
:GoInstallBinaries
```

重新打开vim编辑器，完成插件加载

## 手动配置

安装epel源

```bash
yum install -y epel-release
```

安装编译vim所需的依赖包

```bash
yum install -y gcc git ncurses-devel ctags ruby ruby-devel lua lua-devel luajit luajit-devel ctags git python python-devel tcl-devel perl perl-devel perl-ExtUtils-ParseXS perl perl-devel perl-ExtUtils-ParseXS perl-ExtUtils-Embed
```

下载vim代码

```bash
git clone https://github.com/vim/vim.git $GOPATH/src/github.com/vim/vim
```

编译安装vim

```bash
cd $GOPATH/src/github.com/vim/vim
// 如果不需要vim桌面版UI，可以关闭CONF_OPT_GUI配置项，提示vim性能
sed -i 's/^# CONF_OPT_GUI.*/CONF_OPT_GUI = --disable-gui/' ./src/Makefile
// 编译配置
./configure --with-features=huge \
            --enable-multibyte \
            --enable-rubyinterp=yes \
            --enable-pythoninterp=yes \
            --with-python-config-dir=/lib64/python2.7/config \
            --enable-perlinterp=yes \
            --enable-luainterp=yes \
            --enable-gui=gtk2 \
            --enable-cscope \
            --prefix=/usr/local
// 编译与安装
make VIMRUNTIMEDIR=/usr/local/share/vim/vim81 && make install
```

在~/.vimrc路径上创建vim配置文件

{% code title="~/.vimrc" %}
```text
set number
syntax on
"set softtabstop=2
"set tabstop=2
set expandtab
set backspace=2
set incsearch
set hlsearch
set autowrite
set nobackup
set noswapfile
" code fold by indent
set fdm=indent
set pastetoggle=<F9>

" This enables us to undo files even if you exit Vim.
if has('persistent_undo')
  set undofile
  set undodir=~/.config/vim/tmp/undo//
endif

" Use space to close and open code fold
nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>

set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

Plugin 'VundleVim/Vundle.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'Xuyuanp/nerdtree-git-plugin'
Plugin 'fatih/vim-go'
" Plugin 'Valloric/YouCompleteMe' 
" Plugin 'vim-syntastic/syntastic'
" Plugin 'rjohnsondev/vim-compiler-go'
Plugin 'majutsushi/tagbar'
Plugin 'fatih/molokai'
" Plugin 'Shougo/neocomplete.vim'

call vundle#end()
filetype plugin indent on
filetype plugin on

" nerdtree conf
let NERDTreeShowHidden=1
autocmd vimenter * NERDTree
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
autocmd VimEnter * if argc() == 1 && isdirectory(argv()[0]) && !exists("s:std_in") | exe 'NERDTree' argv()[0] | wincmd p | ene | endif
autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif

" molokai conf
syntax enable
set t_Co=256
let g:rehash256 = 1
let g:molokai_original = 1
colorscheme molokai

" vim-go
let g:go_list_type = "quickfix"
let g:go_highlight_functions = 1
let g:go_highlight_methods = 1
let g:go_highlight_structs = 1
let g:go_highlight_operators = 1
let g:go_highlight_build_constraints = 1
"autocmd BufWritePost,FileWritePost *.go execute 'GoMetaLinter'
"autocmd BufNewFile,BufRead *.go setlocal noet ts=2 sw=2 sts=2
"autocmd BufWritePost,FileWritePost *.go execute 'GoVet'

" tagbar
"autocmd vimenter * TagbarToggle
nmap <F8> :TagbarToggle<CR>

" neocomplete
" let g:neocomplete#enable_at_startup = 1
```
{% endcode %}

安装vim插件管理工具

```bash
git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

下载golang插件代码

```bash
git clone git@github.com:golang/net.git $GOPATH/src/golang.org/x/net
git clone git@github.com:golang/sync.git $GOPATH/src/golang.org/x/sync
git clone git@github.com:golang/tools.git $GOPATH/src/golang.org/x/tools
git clone git@github.com:golang/lint.git $GOPATH/src/golang.org/x/lint
```

打开vim编辑器并加载插件

```text
vim -c PluginInstall -c GoInstallBinaries
```

## 快捷命令

| 命令 |  作用 |
| :--- | :--- |
| grep {{ 关键字 }} {{ 路径 }} | 在路径中搜索具有指定关键字的文件位置 |
| set ff={{ 格式 }} | 设置当前打开文件的的格式，可选：unix和dos |

## 快捷键

| 键位 | 作用 |
| :--- | :--- |
| ctrl+w+w | 将光标切换到下一个窗口 |
| ctrl+w+h | 将光标切换到第一个窗口 |
| ctrl+o+x | 自动列出可用字段与方法 |
| ctrl+\] | 从引用处跳转到定义处 |
| ctrl+t | 从定义处跳转回引用处 |
| z+c | 内容折叠 |
| z+o | 展开折叠 |

## Q&A

### 如何修改文件格式？

在windows下编辑过的文件传输到linux上后执行时出现类似错误

```text
/bin/bash^M: bad interpreter: No such file or directory
```

可以通过`:set ff=unix`保存为linux所能识别的格式，可以通过`:set ff=dos`再重新保存回windows所能识别的格式

### 如何避免粘贴时自动缩进和注释？

在粘贴前使用指令`set paste`进入粘贴模式

在粘贴后使用指令`set nopaste`退出粘贴模式

也可以将该指令加入快捷键中

{% code title="~/.vimrc" %}
```text
...
set pastetoggle=<F9>
```
{% endcode %}



