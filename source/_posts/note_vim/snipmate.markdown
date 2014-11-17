title: 好插件之snipmate
Status: Public
Tags: plugin
date: 2013-8-28 11:17:55
---

[TOC]

# 标准好插件

之前因为没有搞定neocomplcache，所以neosnippet也就没有动作了。

但是后来在网上逛来逛去，还是隐隐的觉得还是应该用个snippet插件的，虽然之前没接触过。没用上neosnippet，但是不能否定snippet这种从gif截图功能就能看出牛逼之处的功能。

snippet的功能其实还是很明显的，不过陌生的东西还是会让人有莫名的感觉。这次我选了个用的人最多的snipmate,据说是移植的TextMate的功能，所以这个功能性还有操作应该都是标准的。

<!--more-->

不过安装了之后还是很让人惊喜，因为，**不用任何配置就能用**，然后**用法非常的明显简单**，插入模式下打几个字然后按Tab展开。单看这些就感觉不愧是排行榜上的牛逼插件。

# snippets怎么用

snippet就是代码片段，snipmate会根据代码关键字来展开格式化的代码段，就像示例里面的你在编辑器里面写一个`for`，然后按个Tab，就会展开成：
```c
for (i=0; i<count; i++)
{
	/*arguments*/
}
```
然后如果你要需要的参数不是`i`、`count`可以按Tab到修改点修改。这样写起代码来就行云流水了，不必做无谓的劳动还减少出错概率。想想要是拿这个写verilog就爽的很。

snipmate插件有个snippets文件夹，里面的是一个个以语言名字命名的.snippets文件，这些文件里面就是这些语法snippet的定义了。我看了下这个里面没有verilog.snippets文件。不过这个没关系，snipmate继承了snippet语法自定义，这个才是这个插件的语法自定义。而且，仔细想想的话其实应该是每个人都应该自己写snippet，因为每个人写代码的习惯都不一样，写出最合适自己的snippet才是王道。这样看来snippets文件夹里面放几个示例文件其实也就够了。

来看这段snippet代码：
```
# a farbox header snip
snippet til
	Title: ${1}
	Status: ${2:Public}
	Tags: ${3}
	Date: ${4}

	[TOC]
```
这个是给FarBox的markdown写的文件头，一眼看去一目了然。用的话就在插入模式下打'til'，按Tab展开，接着就是一个个按Tab编辑了。

snippet的语法非常的简洁直观，'#'是注释符，snippet的定义是：
```
snippet name_of_snippet [description of this snippet]
	argument_of_snippet
```
`[description of this snippet]`是可选内容，不过snippet是可以重名的，这个时候就是按`[description of this snippet]`来区分的。

`argument_of_snippet`里面就是snippet的内容，普通字符就是简单的复制。`${\d[:placehold]}`代表一个Tab编辑点，`\d`表示编号，从1开始，可以不按位置排序，`[:placehold]`是可选的默认字符，不想改这个的话按Tab就跳过了。Tab可以按shift反向跳转，这个很好用，这么几个特质加起来写verilog的话基本已经打败了Emacs的verilog-mode里面除了auto-command的其他快速完成的功能了。

snipmate的snippet里面可以用\`vim_function\`执行Vim的函数，这个不知道是不是Vim的私货，像是:
```
snippet time
	`strftime("%c")`
```
这个snippet可以插入时间帧，`time--><Tab>-->2013-8-24 21:19:38`。这些算是锦上添花了。

# 小小的不足

总的说来，snipmate是我目前用过的最好的插件，很是完美。不过bug暴露的很快，之前网上就看到说连续的反向单引号是有bug的。我写了markdown里面的github代码段snippet：
```
snippet code
	```${1:verilog}
	${2}
	\```
```
一运行果然就报错了，然后只出来verilog的字样。这个估计是跟两个反引号调用函数有关系，不过目测应该算是小bug。但是snipmate好久都没更新了，不知道这个什么时候会修复。

然后是网上说的不能立即更新的问题，这个要是你在调试一些snippet的时候是挺不爽的。不过我看了看文档——这个文档也是简短的很，里面有说到`ReloadAllSnippets()`这个函数，然后我就在写snippet的时候试了下`:call ReloadAllSnippets()`这个命令，结果是snippet是可以马上就刷新的。有了这个办法那就好办了，绑个键什么的就很方便了。

这么说snipmate的不足其实只是有反单引号的那个bug。不过连续的反单引号在代码里面基本是不会用到的，这个也就基本可以忽略了。

-----

我后来有在另外的一台电脑上下载了一个叫Ultisnippet的插件想用用，因为这个说的是snipmate的升级，兼容snippet语法，支持嵌套snippet。我就想这个是不是就能解决像是`switch-case`这种可以有任意条的代码段。然后我就用了啊，结果说的是需要安装python。我其实比较怕麻烦，而且我一想这种需要在python上面运行的东西，效率是要直追neocomplcache啊，我的笔记本怕是承受不起了。

这下我要更坚定的拥护snipmate了。




