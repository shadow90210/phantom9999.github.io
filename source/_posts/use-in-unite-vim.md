---
title: unite.vim插件使用及配置
categories: essay
comments: false
abbrlink: b0d02bc6
date: 2018-07-14 16:16:00
updated: 2018-07-14 16:16:00
tags: [ vim, unite.vim, vim插件, vim配置 ]
keywords: vim配置, vim插件, unite.vim配置, unite.vim使用, unite.vim配置
description: 本文主要介绍unite.vim的使用与配置. unite.vim是Shougo开发的一款插件, 确切的说, unite就像vim插件里面的vim. 这个插件功能繁多, 配置项目较多, 一定程度上怎么了用户的学习成本, 本文将详细介绍这个工具的使用与配置.
---

# 简介
本文主要介绍unite.vim的使用与配置.
unite.vim是Shougo开发的一款插件, 确切的说, unite就像vim插件里面的vim.
这个插件功能繁多, 配置项目较多, 一定程度上怎么了用户的学习成本, 本文将详细介绍这个工具的使用与配置.


# 功能介绍
本文介绍的unite.vim的功能包括unite.vim集成的功能以及第三方插件的功能. 
unite.vim由于其丰富的功能以及高扩展性, 已经衍生出了多款基于unite.vim的插件.

unite.vim集成的功能包括:
- bookmark 书签功能
- buffer, buffer_tab buffer浏览功能
- change 列举变动
- command ex命令
- directory, directory/new, directory_mru 目录相关功能
- fie, file/new, file_point, file_rec 文件相关功能
- function 函数相关功能
- grep
- history
- jump, jump_point 跳转相关功能
- launcher
- line, line/fast 行号相关功能
- mapping
- menu
- neocomplete
- output
- process
- resume
- runtimepath
- source
- tab
- vimgrep
- window


基于unite.vim的插件包括:
- unite-help
- unite-tag
- unite-outline
- unite-colorscheme
- unite-font
- unite-locate
- unite-everything
- unite-mark
- unite-alias
- unite-script
- unite-git_grep
- unite-remotefile
- unite-neco
- unite-rake
- unite-history
- unite-qflist
- unite-gem
- unite-qf
- unite-session
- unite-svn
- unite-rails
- unite-grails
- unite-cake
- unite-zf, unite-sf2
- unite-ack
- unite-launch
- unite-transparency
- quicklearn
- vim_hacks
- haskellimport
- unite-equery
- unite-file-vcs
- unite-radio.vim
- unite-gist
- vim-unite-id
- unite-ref


# buffer功能介绍与配置
vim中的buffer类似于ide中已经打开的文件, vim将已经打开的文件保存到buffer中, 方便用户去使用.
unite.vim内置buffer功能, 使用<code>:Unite source</code>命令可以看到.
unite.vim提供的buffer功能包括:
- buffer
- buffer_tab

其中执行<code>:Unite buffer</code>命令后, 会在新的tab中显示buffer列表, 而执行<code>:Unite buffer_tab</code>后会在当前tab中显示buffer.
unite为buffer选择提供了即时搜索的功能, 用户可以搜索关键词, 然后unite查找buffer对应的文件, 然后进行排序. unite查找的内容仅限于路径和文件名.

vim中有类似功能的插件包括, MinBufExplorer和bufexporer插件.

bufexporer插件使用简单, 它提供三个命令分别是<code>\be</code>(打开历史文件列表), <code>\bv</code>(水平创建一个tab显示buffer信息), <code>\bs</code>(垂直创建一个tab显示buffer信息).
这个插件不需要配置, 加载即可使用. 比较麻烦的是, 快捷键比较逆天, 而且不支持buffer的搜索.

MinBufExporer会开一个狭小的tab显示buffer列表信息.
使用minBufExporer方面, minbufexporer跟bufexporer一样, 不需要配置, 可以直接使用.
在minBufExporer使用<code>:bn</code>(下一个buffer), <code>:np</code>(上一个buf), <code>:b"num"</code>,
<code>:MiniBufExporer</code>(打开tab, 并显示buffer信息), <code>:CMiniBufExporer</code>(关闭buffer的tab).

与这两个插件相比, unite buffer显得无比强大好用.











# 文件查找功能介绍与配置

# outline功能介绍与配置


