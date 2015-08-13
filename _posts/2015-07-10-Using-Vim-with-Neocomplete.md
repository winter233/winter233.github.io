---
layout: post
title: Using Vim with Neocomplete on Ubuntu 14.04
categories: vim
tags: [linux,vim,ubuntu]
---

[Neocomplete](https://github.com/Shougo/neocomplete.vim) is an auto completion plugin for vim. However, it requires [lua](http://www.lua.org), which is not supported by vim default. You can easily install Vim by opening "Ubuntu Software Center", then search for `Gvim`, then click install. Here is a guide to compile Vim with `lua` support from source. It was originally written by [Valloric](https://github.com/Valloric/), the author of [YouCompleteMe](https://github.com/Valloric/YouCompleteMe). The reason why I don't use `YouCompleteMe` is that I have never built it successfully :-( 

1. Install prerequisite libraries:    

    ```sh
    sudo apt-get install libncurses5-dev libgnome2-dev libgnomeui-dev \
        libgtk2.0-dev libatk1.0-dev libbonoboui2-dev \
        libcairo2-dev libx11-dev libxpm-dev libxt-dev python-dev \
        ruby-dev mercurial
    ```

2. Remove original Vim:     

    ```sh
	sudo apt-get remove vim vim-runtime gvim
    ```

3. Install lua:

	```sh
	sudo apt-get install liblua5.2-dev lua5.2
	```

4. Compile Vim74 from source:

	```sh
	wget ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2
	tar xvf vim-7.4.tar.bz2
	cd vim74
	./configure --with-features=huge \
            --enable-multibyte \
            --enable-rubyinterp \
            --enable-pythoninterp \
            --with-python-config-dir=/usr/lib/python2.7/config \
            --enable-perlinterp \
            --enable-luainterp \
            --enable-gui=gtk2 --enable-cscope --prefix=/usr
	```
	set Vim RUNTIMEDIR:

	```sh
	make VIMRUNTIMEDIR=/usr/share/vim/vim74
	```
	Finish install using `sudo make install`.
	To make it easier when uninstall, use `sudo checkinstall` instead.

5. Set Vim as default editor:

	```sh
	sudo update-alternatives --install /usr/bin/editor editor /usr/bin/vim 1
	sudo update-alternatives --set editor /usr/bin/vim
	sudo update-alternatives --install /usr/bin/vi vi /usr/bin/vim 1
	sudo update-alternatives --set vi /usr/bin/vim
	```

6. Check installation by typing `vim --version` in shell or `:echo has("lua")` in vim.

Here are some common settings:

```vim
nmap <F3> :NeoCompleteToggle<CR>                    "toggle plugin using F3
let g:neocomplete#enable_at_startup = 1             "start automatically
let g:neocomplete#disable_auto_complete = 0         "auto completion
let g:neocomplete#auto_completion_start_length = 3  "do completion after input 3 keys
```

More info can be found by `:help neocomplete`.   
Here is my [vimrc](https://github.com/winter233/tools-conf/blob/master/.vimrc) file.

Ref: [Building Vim from source](https://github.com/Valloric/YouCompleteMe/wiki/Building-Vim-from-source)

