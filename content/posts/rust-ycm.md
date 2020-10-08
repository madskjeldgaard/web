---
title: "Setting up Vim and YouCompleteMe for Rust development"
date: 2020-06-11T11:07:41+02:00
draft: false
toc: true
tags:
- vim
- nvim
- rust

---

These are notes for setting up [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe) - a program for autocompletion, code linting and debugging in the text editor Vim - for developing Rust programs. I find this program vital to my Rust workflow because it allows me to more easily explore code and correct mistakes before they reach the compiler. YouCompleteMe will tell you if there are syntactic mistakes in your code, it will tell you about unused variables and code and it will often suggest changes to problematic code. 

## Install the plugin

I am assuming here that you have both `git` and Vundle installed for NVim.

In Nvim, install YouCompleteMe by adding the plugin to your init.vim-file by executing `:e $MYVIMRC`.

Then, after `call plug#begin()` and before  `call plug#end()` add this line:

```vim 
Plug 'ycm-core/YouCompleteMe'
```

Then re-source the init file by running the command `:source $MYVIMRC` and then install the plugin using `:PlugInstall`.

## Install completion
Next up we need to install the completion engines needed. You can cherry-pick the engines you need or do not need but I honestly just install it all.

This is done by going into the plugin folder of YouCompleteMe in a terminal, again assuming you are using Vundle for your plugins this will probably be:

```shell
cd ~/.vim/plugged/YouCompleteMe/
./install.py --all
```

This will take a while to install.

Once that's done, there is a final step which is quite strange and hopefully it will change in the future. According to [the full installation guide](https://github.com/ycm-core/YouCompleteMe/wiki/Full-Installation-Guide), you need to install [rustup](https://www.rustup.rs) if you haven't already.

Then we need to get toolchains and add them to YouCompleteMe.

```shell
# Temporarily redirect rustup to be able to extract the source
RUSTUP_HOME=~/rusttmp

# Get the tools
rustup toolchain install nightly
rustup default nightly
rustup component add rls rust-analysis rust-src
cd $RUSTUP_HOME/toolchains
```
Now you should be in the toolchains folder that you just downloaded. There should be a folder there named something depending on your system (on mine it's `nightly-x86_64-unknown-linux-gnu` because I'm on linux). Move into it using `cd <folder_name>`.

Then, move the contents of the folder of your YouCompleteMe installation `~/.vim/plugged/YouCompleteMe/third_party/ycmd/third_party/rls`. 

Check that you did it correctly by seeing the contents:
```shell
ls ~/.vim/plugged/YouCompleteMe/third_party/ycmd/third_party/rls
```
This should return something like
`bin  etc  lib  share`

## Extra configuration

I am actually not sure this is necessary anymore but I read somewhere that you need to enable Rust features in YouCompleteMe's extra configuration file.

Do this by adding this line to your `init.vim` file:

`let g:ycm_global_ycm_extra_conf = '~/.config/nvim/global_extra_conf.py'`

Then edit the config file in NVim by executing `:tabnew ~/.config/nvim/global_extra_conf.py`.

Here is what mine looks like:

```python
def Settings(**kwargs):
    if kwargs['language'] == 'rust':
        return {
            'ls': {
                'rust': {
                    # 'features': ['http2', 'spnego'],
                    'all_features': True,
                    'racer_completion': True,
                    # 'all_targets': False,
                    # 'wait_to_build': 1500,
                }
            }
        }
```

## Keybindings

Here are the keybindings I use for Rust/YouCompleteMe:

```vim
" YCM KEYBINDINGS
function! YcmStuff() 
    nnoremap si :YcmCompleter GoToDefinition<cr>
    nnoremap sk :YcmRestartServer<cr>
    nnoremap <F1> :YcmCompleter FixIt<cr>
    nnoremap K :YcmCompleter GetDoc<cr>
    nnoremap ; :YcmCompleter GetType<cr>
endfunction

function! Rusty()
    nnoremap <C-e> :terminal cargo run<cr>
    inoremap <C-e> <esc>:terminal cargo run<cr>
endfunction 

augroup rust
    autocmd!
    autocmd FileType rust call Rusty()
	autocmd FileType rust call YcmStuff()
augroup end

```

## UltiSnips

I use these configuration lines to make UltiSnips work with YouCompleteMe to be able to use snippets as well as autocompletion. 

```vim
" YouCompleteMe and UltiSnips compatibility.
let g:ycm_use_ultisnips_completer = 1
let g:ycm_key_list_select_completion=[]
let g:ycm_key_list_previous_completion=[]

" Expand snippets from UltiSnips with tab
let g:UltiSnipsExpandTrigger="<Tab>"
let g:UltiSnipsJumpForwardTrigger="<Tab>"
let g:UltiSnipsJumpBackwardTrigger="<c-tab>"
let g:UltiSnipsSnippetDirectories = ['UltiSnips']
```

