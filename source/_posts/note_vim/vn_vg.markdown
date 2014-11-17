title: Vundle和git
date: 2013-8-28 11:18:05
---

## 前言

Vundle是现在管理vim插件的主流，但是折腾到现在才装起来。Vundle是基于git的一个插件，github简直是vim插件最完美的一个社区，而且当下基本所有的vim插件项目都托管到github了。

我觉得Vundle对像我这种初级用户还是有点距离的，原因主要是对git概念的不甚理解。原先git就是Linux下的工具，像我这种Windows用户知道这个名字是通过github这个很火网站，而且很长的时间里面我一直认为git就是指的github，虽然git跟github的关系是非常大的。

搞清楚git是什么东西花了我很长的时间，然后要用安装使用软件也是有点麻烦，又是有github for windows又是有msysgit。

总之安装好这个Vundle还是花了大力气的，这里就简单的对整个过程做个流水记录，希望能帮到需要的人。

<!--more-->

-----
## 过程

- 准备
	+ 环境：WinXP
	+ 所用工具： mysgit，Vundle(当然是github里面的啦，*Shougo/neocomplcache.vim*)
- git的安装配置，这个我用的是msysgit，因为网上的教程用的都是这个
	- 下载，安装这个没什么好说的，安装完成后桌面上出来一个Git Bash的图标，还是一个命令行的工具，果然是geek的可以。
	- 配置环境变量，不然不能在cmd里面使用git。具体就是把git的安装路径下的bin路径和cmd路径添加到用户变量的PATH字符串里面。然后cmd里面查看版本：git --version就会出现
`git version 1.8.1.msysgit.1`
证明配置成功了。这里要说的是，在Git Bash里面打“git --version”查看版本是可以的，但是还是要把这个路径放入环境变量。
	- 在git的cmd文件夹下建立curl脚本(curl.cmd)，用于“远程链接”，脚本内容为为：

```
@rem Do not use "echo off" to not affect any child calls.
@setlocal
@rem Get the abolute path to the parent directory, which is assumed to be the
@rem Git installation root.
@for /F "delims=" %%I in ("%~dp0..") do
@set git_install_root=%%~fI
@set PATH=%git_install_root%\bin;%git_install_root%\mingw\bin;%PATH%
@if not exist "%HOME%" 
@set HOME=%HOMEDRIVE%%HOMEPATH% 
@if not exist "%HOME%" 
@set HOME=%USERPROFILE% 
@curl.exe %*
```
其实这个东西的意义我是看不懂的，直接照着做。这里要说的是教程有的是说“git的路径放入环境变量”，有的是说“把bin路径和cmd路径都放入环境变量”，而且教程里面没有明说这个curl.cmd是要放在哪个文件夹下面，我没多想，把这个文件放cmd文件假下面了，然后把两个都放进PATH里面去了。操作完之后还是要在cmd里面验证一下curl，输入“curl --version”查看版本，输出`curl 7.26.0 (i686-pc-mingw32) libcurl/7.26.0 OpenSSL/0.9.8x zlib/1.2.7`那就对了。

- Vundle的安装配置
	- 我发现网上教程说的安装基本都是：
`git clone http://github.com/gmarik/vundle.git ~/.vim/vundle.git`
~/.vim后面的路径是稍有不同，搞的我已经会用git了似的。这个其实是Vundle doc里面的setup的第一步，但是像我虽然不懂git，这个路径是linux的路径还是看的出来的。而且根据我非常有限的linux知识，“~/”代表的是HOME目录，说明这个命令行还是在vim里面执行的。唉，但是我这边的vim里面执行不来git命令。所以我只好用土法，去github的gmarik/vundle把zip包下过来，然后在vimfiles文件夹下面创建Bundle文件夹然后把解压后的包拷进去。这里有另外一个奇怪的地方，就是vundle.txt里面的这个命令行路径是，~/.vim/vundle.git但是markdown文档里面的setup给的路径是~/.vim/bundle/vundle，我也管不了这么多了。
	- 然后就是_vimrc文件的配置了：

			set nocompatible
			" Vundle configs
			filetype off 
			" set the path of Vundle
			set rtp+=$VIM/vimfiles/bundle/vundle/ 
			call vundle#rc('$VIM/vimfiles/bundle/') 
			""call vundle#rc() 
			" set the protocol 'git' instead of 'http'
			let g:vundle_default_git_proto = 'git' 
			Bundle 'gmarik/vundle' 
			filetype plugin indent on
			" My Bundle list
			Bundle 'taglist.vim' 
			Bundle 'scrooloose/nerdtree'
			Bundle 'Shougo/neocomplcache.vim'   
我是把这些放在_vimrc的最前头了，文档里面一再强调配置前“filetype off”的和配置后的“filetype plugin indent on”，然后保险起见在最后一行又加了一次“filetype plugin indent on”。

- 最后就讲讲使用情况
	- _vimrc文件里面用“Bundle 'namelink_of_plugin'”这样的格式来标记需要用Vundle管理的插件，支持3种格式的namelink_of_plugin：github的vimscript主页（也就是vim官方插件）下的比如'taglist.vim'；github的非vimscript主页的插件比如'scrooloose/nerdtree'；还有就是不在github下的插件了，要打完连接全称，这个我估计也用不到了。
 - 命令很少，一共也就几条：:BundleList——列出插件；:BundleInstal(!)——安装（更新）插件；:BundleSearch——搜索插件，要带插件名的；:BundleClean——删除插件。
	- 用:BundleList命令会自动打开一个左侧的window，里面是_vimrc里面的Bundle关键字插件。
	- :BundleInstal(!)我用的是加了叹号的更新插件操作，这个用了会跳出cmd的窗口，估计就是在cmd里面使用一系列的git命令。这个同样会在vim左侧开出一个window来，有运行状态的跟踪，整个过程完了之后会提示你点*l*键查看log，蛮不错的。不过在这里是有出了状况了。第一个就是Vundle的更新，会一直失败，然后log里说的大致是"Vundle directory already exists"什么的，我想这可能跟我是用土办法直接拷进去有关系，这个时候我把Vundle这个文件夹删掉了才解决了这个错误。**注意这个时候我是开着vim删文件夹的**，要是关了vim然后删了再重启估计会有问题。我想这个Vundle的这个设置其实有点莫名其妙啊，会不会是个bug。第二个就是taglist这个插件了，会提示"repo does not exist"，搞了半天才醒悟是因为完整的插件名其实是'taglist.vim'。还有要说的是，其实我是已经安装了taglist跟NERDTree的，是直接拷到vimfiles这个文件夹的，然后用了这个命令，Vundle是在vimfiles/Bundle下面把这两个插件重新安装了，然后neocomplcache这个是完全要Vundle安装的。
	- :BundleSearch这个真的比较犀利啊，输入字符搜插件。也是打开左侧窗口，然后列出搜索结果，然后会提示你安装啊更新啊清理啊这些后续操作。
	- :BundleClean我还没用过，不过我猜测这个的原理可能是比较_vimrc文件里的Bundle项目跟Bundle文件夹，发现_vimrc里面没有的就把Bundle里面的删了。不过Vundle是在Bundle下面为每个插件建立一个独立的目录而不是像我之前的直接把解压的所有东西都丢到vimfiles里面，然后很多插件都会在同一个目录比如plugin、autoload、ftplugin里面混到一起去。我原先要用Vundle就是冲着插件清理的功能来的，我想要是真能把混到一起的文件找出来然后删掉还是很犀利的。Vundle没有让我看到想看到的东西，不过这个设计还是巧妙的。

## 小总结

总的说来，Vundle是一个好东西，一番折腾之后还是有点快感的。之前比较早的时候，我在网上看到说Vundle可以让_vimrc少很多行，我当时肯定是看错了。。。








