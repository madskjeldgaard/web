---
title: NeoVim setup for c++ and openFrameworks development
author: mads
type: post
date: 2020-04-06T15:44:23+00:00
url: /neovim-setup-for-c-and-openframeworks-development/
featured_image: /wp-content/uploads/2020/04/2020-04-06-174329_1920x1080_scrot.png
post_grid_post_settings:
  - 'a:14:{s:9:"post_skin";s:4:"flat";s:19:"custom_thumb_source";s:101:"https://www.madskjeldgaard.dk/wp-content/plugins/post-grid/assets/frontend/css/images/placeholder.png";s:16:"thumb_custom_url";s:0:"";s:17:"font_awesome_icon";s:0:"";s:23:"font_awesome_icon_color";s:0:"";s:22:"font_awesome_icon_size";s:0:"";s:17:"custom_youtube_id";s:0:"";s:15:"custom_vimeo_id";s:0:"";s:21:"custom_dailymotion_id";s:0:"";s:14:"custom_mp3_url";s:0:"";s:20:"custom_soundcloud_id";s:0:"";s:16:"custom_video_MP4";s:0:"";s:16:"custom_video_OGV";s:0:"";s:17:"custom_video_WEBM";s:0:"";}'

---
It is possible to get a nice development environment on Linux (and other platforms) using NeoVim and a few plugins and settings.

This dev environment includes snippets, autocomplete, debugging and smart code suggestions for methods.

I got a lot of pointers for this setup from [Chendi Xue&#8217;s blogpost about Vim/CPP development][1].

So, without further ado here are my notes for setting up shop using YouCompleteMe, UltiSnips and some formatting plugins.

## Install clang formatting

On Arch Linux I had to install clang to make this work: `sudo pacman -Syu clang`

There&#8217;s some really nice autoformatting plugins from Google. Add these to your init.vim and install:

    " Code formatting (for c++)
    Plug 'google/vim-maktaba'
    Plug 'google/vim-codefmt'
    Plug 'google/vim-glaive'
    

This should now automatically fix your code automatically with the following in your init.vim:

    " CPP setup done using this tutorial https://xuechendi.github.io/2019/11/11/VIM-CPP-IDE-2019-111-11-VIM_CPP_IDE
    " Code formatting
    autocmd FileType c,cpp,proto,javascript AutoFormatBuffer clang-format
    

## Ctags

Ctags makes it possible to autocomplete source code when you are writing code and with the power of YCM it should also make it possible to work with methods of classes in a nice way.

First, install Ctags
  
Arch: `sudo pacman -Syu ctags`

To compile tags files for my open frameworks projects I use the python tool [compiledb][2]

This can be installed using pip:
  
`pip install compiledb`

Now, from the root of your open frameworks projects you can run the following command to generate a tags file:

`compiledb -n make`

This will output a file called `compile_commands.json` containing the tags.

## YouCompleteMe

[<img class="alignnone size-full wp-image-874" src="https://www.madskjeldgaard.dk/wp-content/uploads/2020/04/of-ycm.gif" alt="" width="953" height="1004" />][3]

Following this installation guide thoroughly [installation guide][4]

Especially make sure you have [clangd][5] installed. On arch, this was installed automatically for me with the `clang` package but you can verify this on the command line by running `which clangd` which should return a path to the binary, otherwise you need to install it.

It&#8217;s especially important to add this line to your init.vim file:

    <br />let g:ycm_clangd_binary_path = "/path/to/clangd"
    
    

`/path/to/clangd` can be replaced by the output of the command `which clangd`

Now you should have automatic debugging and autocomplete suggestions.

Try opening up a openframeworks, run the `compiledb -n make` command and move to `ofApp.cpp` and inside the setup function add the following:

`auto rect = ofRectangle(0, 0, 250, 250);`

This will make a rectangle object. Now, in the next line, write `rect.` &#8211; by the `.` you should now see a list of suggested methods to use that are native to the ofRectangle class.

## Keybindings

Most of the relevant keymappings I use, I got from [Kenneth Flak&#8217;s blogpost about the same][6]. These will setup vim to compile your oF program when you press ctrl-e (this is convenient since this is the same shortcut we use in [SCNVim][7]):

    augroup c
    autocmd!
    autocmd FileType c,cpp,h,hpp,glsl call MakeRun()
    augroup end
    
    function! MakeRun()
    nnoremap :terminal make -j8 && make run
    inoremap :terminal make -j8 && make run
    endfunction
    

And then some nice keybindings to work with YouCompleteMe:

    " Find definition
    au FileType cpp nnoremap si :YcmCompleter GoToDefinition
    
    " Reboot Ycm server
    au FileType cpp nnoremap sk :YcmRestartServer
    
    " Fix thing under cursor
    au FileType cpp nnoremap :YcmCompleter FixIt
    
    " Regenerate tags file
    au FileType cpp nnoremap :!compiledb -n make
    
    " Go to documentation
    au FileType cpp nnoremap K :YcmCompleter GetDoc
    
    " Echo the type/arguments of class
    au FileType cpp nnoremap ; :YcmCompleter GetType
    
    " new split with alternate file
    nnoremap mv :AV
    " switch in same window
    nnoremap ma :A
    

# Snippets

[<img class="alignnone size-full wp-image-873" src="https://www.madskjeldgaard.dk/wp-content/uploads/2020/04/of-snippets.gif" alt="" width="953" height="1004" />][8]
  
Another really nice feature in this dev environment is the addition of snippets. I use UltiSnips for this. When I set this up, I read varying reports about how well UltiSnips works with YCM, but for me it works fine.

Install [UltiSnips][9] and then add these to your init.vim:

    " YouCompleteMe and UltiSnips compatibility.
    let g:ycm_use_ultisnips_completer = 1
    let g:ycm_key_list_select_completion=[]
    let g:ycm_key_list_previous_completion=[]
    
    " Expand snippets from UltiSnips with tab
    let g:UltiSnipsExpandTrigger=""
    let g:UltiSnipsJumpForwardTrigger=""
    let g:UltiSnipsJumpBackwardTrigger=""
    
    " Where the different snippet directories are stored
    let g:UltiSnipsSnippetDirectories = ['UltiSnips', 'scnvim-data', 'plugged/vim-snippets']
    

I&#8217;d also recommend installing [vim-snippets][10] which contains extra snippets.

Now, when for example you type `if` in a `.cpp` or `.h` file you can hit tab to expand it to get the full code for an if-statement. If you want, you can even add your own custom snippets using Ultisnips powerful capabilities ( execute `:h snippets` in Vim for more info about this ). Below I have defined two snippets. One which will expand when typing `ofclass` and one when typing `ofheader`. The first will create a sensible class and the latter a sensible header. Hit tab to cycle through the arguments.

To use these snippets, add these to the file (which you can create if it doesn&#8217;t exist yet) `~/.vim/UltiSnips/cpp.snippets`

    snippet ofclass "Class for open frameworks"
    #include "${1:`!p snip.rv = snip.basename`}.h"
    
    $1::$1(){}
    
    void $1::setup(){${2:}}
    
    void $1::update(){${3:}}
    
    void $1::draw(){${4:}}
    endsnippet
    
    snippet ofheader "Header for open frameworks class"
    #ifndef ${1:`!p snip.rv = "_" + snip.basename.upper()`}
    #define $1
    #include "ofMain.h"
    class ${2:`!p snip.rv = snip.basename`} {
    
    public:
    
    void setup();
    void update();
    void draw();
    
    $2(); // constructor
    
    ${0}
    
    private:
    };
    #endif
    endsnippet

 [1]: https://xuechendi.github.io/2019/11/11/VIM-CPP-IDE-2019-111-11-VIM_CPP_IDE
 [2]: https://pypi.org/project/compiledb/
 [3]: https://www.madskjeldgaard.dk/wp-content/uploads/2020/04/of-ycm.gif
 [4]: https://github.com/ycm-core/YouCompleteMe/wiki/Full-Installation-Guide
 [5]: https://clangd.llvm.org/installation.html
 [6]: http://roosnaflak.com/tech-and-research/openframeworks-nvim-archlinux/
 [7]: https://github.com/davidgranstrom/scnvim
 [8]: https://www.madskjeldgaard.dk/wp-content/uploads/2020/04/of-snippets.gif
 [9]: https://github.com/SirVer/ultisnips
 [10]: github.com/honza/vim-snippets/