## 安装tagbar
安装tagbar需要先安装ctag， 在ubuntu下面的暗转配置为:
    
    sudo apt-get install ctags

然后对tagbar进行配置， 在使用vim+go的时候， 只需要安装tagbar插件：并添加配置即可如下:
    
    cd ~/.vim/bundle
    git clone https://github.com/majutsushi/tagbar.git
    
在配置文件中添加以下的内容，然后正常打开文件， 就会自动生成tag


    let g:tagbar_type_go = {
        \ 'ctagstype' : 'go',
        \ 'kinds'     : [
            \ 'p:package',
            \ 'i:imports:1',
            \ 'c:constants',
            \ 'v:variables',
            \ 't:types',
            \ 'n:interfaces',
            \ 'w:fields',
            \ 'e:embedded',
            \ 'm:methods',
            \ 'r:constructor',
            \ 'f:functions'
        \ ],
        \ 'sro' : '.',
        \ 'kind2scope' : {
            \ 't' : 'ctype',
            \ 'n' : 'ntype'
        \ },
        \ 'scope2kind' : {
            \ 'ctype' : 't',
            \ 'ntype' : 'n'
        \ },
        \ 'ctagsbin'  : 'gotags',
        \ 'ctagsargs' : '-sort -silent'
        \ }

    nnoremap <C-t> :TagbarToggle<CR>

    let tagbar_ctags_bin="/usr/bin/ctags"


