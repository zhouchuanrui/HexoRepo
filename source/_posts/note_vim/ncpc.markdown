title: neocomplcache这个插件。。。
Tags: plugin
date: 2013-8-28 11:16:59
---

这几天一直在看neocomplcache的文档，在慢慢的试用这个插件，这个感觉非常的痛苦啊。

其实我需要的只是一个比较方便的、稍微智能一点的自动补全工具而已，网上一搜就出来什么SuperTab、OmniCppCompl看的人眼花缭乱，评价还都不怎么样。选来选去选了呼声比较高的neocomplcache这个。其实我是neosnippet和neocomplcache一起下过来的，原先我以为neosnippet就是带有自动补全功能而且升级到了snippets工具的一个强悍插件。只是以我菜鸟的身份不能这么轻举妄动啊，我得先看文档啊，然后粗粗的看了两个插件的doc发现neosnippet这个是没有词补全功能的，还是得要neocomplcache这个来。

<!--more-->

但是没办法，无论是文档还是实际使用，neocomplcache都给我带来了很大的痛苦，其实可能使用不好是我没有好好读文档的原因，而没有好好读文档可能是文档不好的原因，文档不好可能是设计不好。。。
我就来说说为什么neocomplcache这么让我痛苦把。

-----

### 插件设计
> <iles/bundle/neocomplcache.vim/
|~autoload/
| |~neocomplcache/
| | |+filters/
| | |+sources/
| | |-async_cache.vim
| | |-cache.vim
| | |-commands.vim
| | |-complete.vim
| | |-context_filetype.vim
| | |-filters.vim
| | |-handler.vim
| | |-helper.vim
| | |-init.vim
| | |-mappings.vim
| | |-util.vim
| | \`-variables.vim
| |+unite/
| |+vital/
| |-neocomplcache.vim
| \`-vital.vim
|~doc/
| |-neocomplcache.txt
| \`-tags
|~plugin/
| |~neocomplcache/
| | |-buffer_complete.vim
| | |-dictionary_complete.vim
| | |-include_complete.vim
| | |-syntax_complete.vim
| | \`-tags_complete.vim
| \`-neocomplcache.vim
|~vest/
| \`-test-neocomplcache.vim
\`-README.md

这个就是neocomplcache的目录，有几个文件夹还没打开呢，完全打开的话肯定是非常的犀利。这个是我目前为止见过的最为复杂的一个插件。这么复杂的东西在提供犀利功能的同时bug肯定也不少吧，这是我的看法。再加上网上十分常见的关于neocomplcache跟其他插件冲突的传闻，就更让我觉得这个设计肯定有问题啊，虽然不能说是neocomplcache单方面的问题。

说起来，我能比较坚决的想要去用neocomplcache的一个很大的原因是我没有用这些跟它有冲突的插件。

### 文档

文档给人的感觉还是大，有将近2000行，其中FAQ就有300多行。

里面command这一节给人的感觉还是中规中矩，也都是一些简单直观的操作。这里要说一说里面的一个奇葩命令：

    :Neco [{anim-number}]				*:Neco*
		Secret.
一打这个命令会在命令窗口出一段卖萌字符动画，但是想我现在这种状态简直就没心情欣赏这些。

其实说command这部分中规中矩不能算很客观，这对其他插件来说不太公平，命令是有些多了，尤其是里面有还有:NeoComplCacheClean这种暗示你neocomplcache其实需要你手动清理缓存的命令。

这个中规中矩是相对而言的，你看了variables这一节就会有这种感觉了。这一节对我的冲击力真是太强悍了，主要表现为：

1. 这个文档的一半内容都是variables这个部分的，可能有上百条；
2. 这些variables明显可以看出不是平行结构而是层级结构的，但是安排的完全没什么条理，我看了下也不是按名字排序下来的啊；
1. 多处出现`"this may be too slow"`、`"...may be too heavy"`这种话，搞的我很无奈啊；
1. 实在是太多条了，我在想，一个插件可以有近百个可以配置的参数，这难道显的很牛逼？

看到后来还有函数，估计是提供API的，然后还有keymapping，然后讲的还是函数，这里实在是看不懂了。

### 配置

以我的水平和现在的状态，也只能在文档的examples里面挑几条出来了事了。

牛逼如neocomplcache果然是连examples也是犀利的很，就像

		" Disable AutoComplPop.
		let g:acp_enableAtStartup = 0
这条，看说明我以为是取消自动补全的popmenu自动跳出的，结果不是这么个效果。唉，搞了半天我才从上面的的注释中看出端倪，这个AutoComplPop会不会是另一个插件，一搜索果然是啊……搞的我郁闷的很。然后我的_vimrc文件的neocomplcache配置就剩下了这么4条：

		let g:neocomplcache_enable_at_startup = 1
		let g:neocomplcache_enable_smart_case = 1
		let g:neocomplcache_min_syntax_length = 3
		let g:neocomplcache_lock_buffer_name_pattern = '\*ku\*'
我决定短时间内不再动弹了。

### 使用体验

不管怎么样还是讲下实际的使用体验，可以说由于我只用了这么4句配置，neocomplcache用起来不怎么给力。其实基本的自动弹出功能是有了，popmenu下来条目非常的多，也能看出有智能排序。但是重要的是，我没有用`"this may be too slow"`、`"...may be too heavy"`这些相关的配置啊，但是用起来怎么还是卡卡的。而且，最为不爽的就是移动光标的时候是会触发补全的，这在写文档有中英文混排的时候，那种感觉，特别的强烈。

我在写这个文档的时候是直接用了:NeoComplCacheDisable这个命令，然后用的默认的补全。现在我发现默认的补全其实也很不错的啊，不就需要自己再按个键吗，呵呵呵呵。

### 小总结

总的说来，不能因为我是个水人，没有毅力读完文件好好的搞出适合自己的配置用不好neocomplcache，就这样否定neocomplcache是个很强悍的作品。但是这个时候我非常的怀念Emacs的Auto-Complete，轻松愉快的完成了智能补全的功能，对比neocomplcache来简直是好的让人无话可说。

我需要的是什么呢？我需要的其实很简单啊，就是打开一个工程目录的里的一个源文件，能够实现整个目录的源文件词补全就可以。Emacs的Auto-Complete可以实现这个之外的所有Elisp系统函数、关键字补全，我需要的没这么多，又比直接Ctags产生的tag文件里面实现所有补全多一点点。

吐槽了这么多其实只是为了释放一下，看看现状还是得继续折腾。

其实我还是非常期待能出现一个全新的、完美的插件能解救被neocomplcache蹂躏的我。



