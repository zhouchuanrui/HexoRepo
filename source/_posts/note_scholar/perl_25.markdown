title: 【译】我与Perl的两个半小时
Status: Public
Tags: Perl
date: 2013-7-5 10:52:11
---

[TOC]

原文: Learn Perl in about 2 hours 30 minutes By Sam Hughes

安装git之后附带装上了Perl, 感觉这个学习成本就一下降了好多. cmd里面打`perl -version`可以查看版本号. 原文的标题是"两个半小时学习(学完)Perl". 这些都有诱惑力. 然后我就决定花点时间学一学, 这篇文档不太长, 翻译一下看看. 希望是有所帮助的.

<!--more-->

-----

Perl is a dynamic, dynamically-typed, high-level, scripting (interpreted) language most comparable with PHP and Python. Perl's syntax owes a lot to ancient shell scripting tools, and it is famed for its overuse of confusing symbols, the majority of which are impossible to Google for. Perl's shell scripting heritage makes it great for writing glue code: scripts which link together other scripts and programs. Perl is ideally suited for processing text data and producing more text data. Perl is widespread, popular, highly portable and well-supported. Perl was designed with the philosophy "There's More Than One Way To Do It" (TMTOWTDI) (contrast with Python, where "there should be one - and preferably only one - obvious way to do it").

Perl是一种动态的, 动态类型, 高级, 脚本(解释型)语言, 通常拿来跟PHP和Python比较. Perl的语法很大一部分源于古老的shell脚本工具, 闻名于大量混乱符号(@, #, $)的运用, 里面的大部分都是Google不到的. 从shell脚本的继承使得Perl能很好的用来写胶水代码(即用来连接其他脚本的程序的脚本). Perl非常适于处理文本数据以及产生更多文本数据. Perl很流行, 可移植性强, 而且平台支持的很好. Perl的设计哲学是"一个任务有多种解决方案", 与Python的"一个任务应该只有一种最适合的方案".

Perl has horrors, but it also has some great redeeming features. In this respect it is like every other programming language ever created.
This document is intended to be informative, not evangelical. It is aimed at people who, like me:

Perl有很多恐怖的地方, 但也有很多很好的补偿特性. 在这个方面, 跟其他的编程语言是一样的.

这篇文档的用意是启发式的而不是教义式的. 它面向的是像我一样的人:

- dislike the official Perl documentation at http://perl.org/ for being intensely technical and giving far too much space to very unusual edge cases
- learn new programming languages most quickly by "axiom and example"
- wish Larry Wall would get to the point
- already know how to program in general terms
- don't care about Perl beyond what's necessary to get the job done.
- 不喜欢[Perl官方文档](http://perl.org/), 因为它有强烈的技术性的, 在不常用的边角料上着墨太多.
- 用示例来最快的学新的编程语言.
- 希望Larry Wall(Perl的作者?)能知道这点.
- 已经有知道怎么大概的编个程.
- 不关心Perl的"完成任务的关键"以外的东西.

This document is intended to be as short as possible, but no shorter.

这篇文档希望能尽可能的短.

# Preliminary notes前面的话

- The following can be said of almost every declarative statement in this document: "that's not, strictly speaking, true; the situation is actually a lot more complicated". If you see a serious lie, point it out, but I reserve the right to preserve certain critical lies-to-children.
- Throughout this document I'm using example print statements to output data but not explicitly appending line breaks. This is done to prevent me from going crazy and to give greater attention to the actual string being printed in each case, which is invariably more important. In many examples, this results in alotofwordsallsmusheduptogetherononeline if the code is run in reality. Try to ignore this.
- 下面的话适用于文档中的所有陈述: "这个严格的说是不对对的; 情况实际上要更复杂"(我囧). 如果你发现了严重的问题, 请指出来, 但我保留这种小孩子说错话的权利.
- 文档通篇, 我用示例print语句来输出数据, 但是没有显式的插入换行符. 这是为我防止我疯掉, 更重要的是让我能更集中的关注每个情况打印出的字符. 在很多例子中, 代码实际运行起来会出现"alotofwordsallsmusheduptogetherononeline". 忽略这些吧.

# Hello world

A Perl script is a text file with the extension .pl.
Here's the full text of helloworld.pl:

Perl脚本是以.pl为后缀的文本文件. 这里是一个`helloworld.pl`的全部文本:

```perl
use strict;
use warnings;

print "Hello world";
```

Perl scripts are interpreted by the Perl interpreter, perl or perl.exe:

perl脚本是用Perl的解释器(perl或perl.exe)来解释的:

```
perl helloworld.pl [arg0 [arg1 [arg2 ...]]]
```

A few immediate notes. Perl's syntax is highly permissive and it will allow you to do things which result in ambiguous-looking statements with unpredictable behaviour. There's no point in me explaining what these behaviours are, because you want to avoid them. The way to avoid them is to put use strict; use warnings; at the very top of every Perl script or module that you create. Statements of the form use foo; are pragmas. A pragma is a signal to perl.exe, which takes effect when initial syntactic validation is being performed, before the program starts running. These lines have no effect when the interpreter encounters them at run time.

The semicolon, ;, is the statement terminator. The symbol # begins a comment. A comment lasts until the end of the line. Perl has no block comment syntax.

几个小贴士. Perl的语法是容错性很高的, 允许你做一些会引起不可预测的行为的模糊的事情. 我不想说这些是什么, 因为你应该避免这些. 方法就是在每个你创建的Perl脚本和模块的顶部都加上`use strict; use warnings;`. 像`use foo;`这种表达式叫做(pragmas)编译指令. 编译指令就是程序运行之前, 让语法验证生效的传递给perl.exe的信号. 在运行时, 解释器碰上了这些就不执行了.

`,;,`这些标点是语句终止符. `#`是注释符, 注释到行尾. Perl没有块注释语法.

# Variables变量

Perl variables come in three types: scalars, arrays and hashes. Each type has its own sigil: $, @ and % respectively. Variables are declared using my, and remain in scope until the end of the enclosing block or file.

Perl的变量有三种类型: 标量, 数组和哈希表, 分别用符号`$`, `@`和`%`表示. 变量用`my`来声明, 作用域是在当前块或者文件.

## Scalar variables

A scalar variable can contain:

标量型变量包括:

- undef (corresponds to None in Python, null in PHP)
- a number (Perl does not distinguish between an integer and a float)
- a string
- a reference to any other variable.
- undef(对应Python的`None`和PHP的`null`)
- 数字(Perl不区分整数和小数)
- 字符串
- 其他变量的引用

```perl
my $undef = undef;
print $undef; # prints the empty string "" and raises a warning

# implicit undef:
my $undef2;
print $undef2; # prints "" and raises exactly the same warning

my $num = 4040.5;
print $num; # "4040.5"

my $string = "world";
print $string; # "world"
```

(References are coming up shortly.)

String concatenation using the . operator (same as PHP):

(引用后面就讲)

字符串拼接用的是`.`操作符(跟PHP一样):

```perl
print "Hello ".$string; # "Hello world"
```

## "Booleans" 布尔型

Perl has no boolean data type. A scalar in an if statement evaluates to boolean "false" if and only if it is one of the following:

Perl是没有布尔型数据的. 在if表达式中标量为逻辑"假"有这么几种情况:

- undef
- number 0
- string ""
- string "0"

The Perl documentation repeatedly claims that functions return "true" or "false" values in certain situations. In practice, when a function is claimed to return "true" it usually returns 1, and when it is claimed to return false it usually returns the empty string, "".

Perl的文档反复的指明函数在特定情况下返回"真"或"假". 实际上, 函数声称会返回"真"的时候返回的多是1, 返回"假"的时候返回的多是空字符串"".

## Weak typing 弱类型

It is impossible to determine whether a scalar contains a "number" or a "string". More precisely, it should never be necessary to do this. Whether a scalar behaves like a number or a string depends on the operator with which it is used. When used as a string, a scalar will behave like a string. When used as a number, a scalar will behave like a number (raising a warning if this isn't possible):

确定一个标量包含了一个数字还是一个字符串是不可能的. 更确切的说, 这也是没必要这么做的. 一个标量的是数字属性还是字符串属性取决于所用的操作符. 当用作字符串的时候, 标量就是字符串; 当用作数字的时候, 就是数字(不能实现的时候会出现警告):

```perl
my $str1 = "4G";
my $str2 = "4H";

print $str1 .  $str2; # "4G4H"
print $str1 +  $str2; # "8" with two warnings
print $str1 eq $str2; # "" (empty string, i.e. false)
print $str1 == $str2; # "1" with two warnings
# The classic error
print "yes" == "no"; # "1" with two warnings; both values evaluate to 0 when used as numbers
```

The lesson is to always using the correct operator in the correct situation. There are separate operators for comparing scalars as numbers and comparing scalars as strings:

要记住的就是根据情况来选用正确的操作符. 标量用来做数字还是字符串是有不同的操作符的:

```
# Numerical operators:  <,  >, <=, >=, ==, !=, <=>, +, *
# String operators:    lt, gt, le, ge, eq, ne, cmp, ., x
```

## Array variables 数组

An array variable is a list of scalars indexed by integers beginning at 0. In Python this is known as a list, and in PHP this is known as an array. An array is declared using a parenthesised list of scalars:

数组就是标量的列表, 索引是0起始的整数. Python里面叫做`list`, PHP里面叫做`array`. 数组是用括号包含的标量序列来声明:

```perl
my @array = (
        "print",
        "these",
        "strings",
        "out",
        "for",
        "me", # trailing comma is okay
);
```

You have to use a dollar sign to access a value from an array, because the value being retrieved is not an array but a scalar:

取数组里面的元素要用`$`符号, 因为数组里面的元素已经是标量了:

```perl
print $array[0]; # "print"
print $array[1]; # "these"
print $array[2]; # "strings"
print $array[3]; # "out"
print $array[4]; # "for"
print $array[5]; # "me"
print $array[6]; # returns undef, prints "" and raises a warning
```

You can use negative indices to retrieve entries starting from the end and working backwards:

可以用负数索引来逆向取出成员:

```perl
print $array[-1]; # "me"
print $array[-2]; # "for"
print $array[-3]; # "out"
print $array[-4]; # "strings"
print $array[-5]; # "these"
print $array[-6]; # "print"
print $array[-7]; # returns undef, prints "" and raises a warning
```

There is no collision between a scalar $var and an array @var containing a scalar entry $var[0]. There may, however, be reader confusion, so avoid this.

To get an array's length:

标量和数组用同样的变量名($var, @var, 还有@var的成员$var[0])是不冲突的. 但是这个会引起混淆, 所以还是尽量避免.

要得到数组的长度:

```perl
print "This array has ".(scalar @array)."elements"; # "This array has 6 elements"
print "The last populated index is ".$#array;       # "The last populated index is 5"
```

The arguments with which the original Perl script was invoked are stored in the built-in array variable @ARGV.

Variables can be interpolated into strings:

原生Perl脚本捕获的参数存放在内置的数组`@ARGV`里.

数组变量可以展开插到字符串里面.

```perl
print "Hello $string"; # "Hello world"
print "@array";        # "print these strings out for me"
```

Caution. One day you will put somebody's email address inside a string, "jeff@gmail.com". This will cause Perl to look for an array variable called @gmail to interpolate into the string, and not find it, resulting in a runtime error. Interpolation can be prevented in two ways: by backslash-escaping the sigil, or by using single quotes instead of double quotes.

小心. 你会把某人的邮箱地址"jeff@gmail.com"放到字符串里面. 这会让Perl去找`@gmail`这个数组, 然后没找到, 这样就会出一个运行错误. 这个可以用两种方法避免: 加反斜杠, 或者加单引号.

```perl
print "Hello \$string"; # "Hello $string"
print 'Hello $string';  # "Hello $string"
print "\@array";        # "@array"
print '@array';         # "@array"
```

## Hash variables 哈希

A hash variable is a list of scalars indexed by strings. In Python this is known as a dictionary, and in PHP it is known as an array.

哈希变量是用字符串来索引的标量列表. Python里面叫字典(dictionary), PHP里面叫array(还是这个?).

```perl
my %scientists = (
        "Newton"   => "Isaac",
        "Einstein" => "Albert",
        "Darwin"   => "Charles",
);
```

Notice how similar this declaration is to an array declaration. In fact, the double arrow symbol => is called a "fat comma", because it is just a synonym for the comma separator. A hash is declared using a list with an even number of elements, where the even-numbered elements (0, 2, ...) are all taken as strings.

Once again, you have to use a dollar sign to access a value from a hash, because the value being retrieved is not a hash but a scalar:

要主要这个定义跟数组很是类似. 实际上, "=>"这个符号是叫做"肥逗号(fat comma)", 因为这个就是取代逗号的. 哈希表是用一组偶数个成员定义的, 第偶数个(0, 2, ...)成员都是字符串.

```perl
print $scientists{"Newton"};   # "Isaac"
print $scientists{"Einstein"}; # "Albert"
print $scientists{"Darwin"};   # "Charles"
print $scientists{"Dyson"};    # returns undef, prints "" and raises a warning
```

Note the braces used here. Again, there is no collision between a scalar $var and a hash %var containing a scalar entry $var{"foo"}.

You can convert a hash straight to an array with twice as many entries, alternating between key and value (and the reverse is equally easy):

要注意这里用到了大括号. 在说一次, 同名的哈希变量跟标量也是不冲突的.

可以直接把哈希变量转换成数组(数组成员会有两倍), 转换键和值(反过来也一样容易):

```perl
my @scientists = %scientists;
```

However, unlike an array, the keys of a hash have no underlying order. They will be returned in whatever order is more efficient. So, notice the rearranged order but preserved pairs in the resulting array:

然而, 跟数组不一样的是键的值是没有预定义的顺序的, 转换后会返回默认最有效率的顺序. 所以, 要注意这个重排次序:

```perl
print "@scientists"; # something like "Einstein Albert Darwin Charles Newton Isaac"
```

To recap, you have to use square brackets to retrieve a value from an array, but you have to use braces to retrieve a value from a hash. The square brackets are effectively a numerical operator and the braces are effectively a string operator. The fact that the index supplied is a number or a string is of absolutely no significance:

回顾一下, 要用方括号从数组中取出一个值, 哈希表的话要用花括号. 方括号实际上是数值操作符, 花括号是字符串操作符. 索引用的是数字还是字符串是是没关系的:

```perl
my $data = "orange";
my @data = ("purple");
my %data = ( "0" => "blue");

print $data;      # "orange"
print $data[0];   # "purple"
print $data["0"]; # "purple"
print $data{0};   # "blue"
print $data{"0"}; # "blue"
```

## Lists 列表

A list in Perl is a different thing again from either an array or a hash. You've just seen several lists:

Perl里面列表是不同于数组和哈希表的存在. 来看几个列表:

```perl
(
	"print",
	"these",
	"strings",
	"out",
	"for",
	"me",
)

(
	"Newton"   => "Isaac",
	"Einstein" => "Albert",
	"Darwin"   => "Charles",
)
```

A list is not a variable. A list is an ephemeral value which can be assigned to an array or a hash variable. This is why the syntax for declaring array and hash variables is identical. There are many situations where the terms "list" and "array" can be used interchangeably, but there are equally many where lists and arrays display subtly different and extremely confusing behaviour.

Okay. Remember that => is just , in disguise and then look at this example:

列表不是变量, 是用来给数组和哈希表赋值的临时值. 这就是为嘛数组和哈希表的定义是一样的. 很多地方"list"跟"array"是可以混用的, 但是很多地方"list"和"array"是表现出完全不同容易混淆的特性.

嗯, 记住"=>"是伪装的, 来看看这个例子:

```perl
("one", 1, "three", 3, "five", 5)
("one" => 1, "three" => 3, "five" => 5)
```

The use of => hints that one of these lists is an array declaration and the other is a hash declaration. But on their own, neither of them are declarations of anything. They are just lists. Identical lists. Also:

"=>"暗示了一个是数组一个是哈希表. 但是这俩都不是什么定义, 它们都只是一模一样的列表. 再来一个

```
()
```

There aren't even hints here. This list could be used to declare an empty array or an empty hash and the perl interpreter clearly has no way of telling either way. Once you understand this odd aspect of Perl, you will also understand why the following fact must be true: List values cannot be nested. Try it:

这连暗示都没了. 这个列表可以用来声明一个空数组或哈希表, Perl解释器也不能分辨这两者. 当知道Perl的这个奇怪特性, 你也就知道了为什么列表的值是不能嵌套的. 看这个:

```perl
my @array = (
	"apples",
	"bananas",
	(
		"inner",
		"list",
		"several",
		"entries",
	),
	"cherries",
);
```

Perl has no way of knowing whether ("inner", "list", "several", "entries") is supposed to be an inner array or an inner hash. Therefore, Perl assumes that it is neither and flattens the list out into a single long list:

Perl不知道`("inner", "list", "several", "entries")`是内置列表还是哈希表, 所以Perl认为这个两者都不是, 将其展开成一个长列表:

```perl
print $array[0]; # "apples"
print $array[1]; # "bananas"
print $array[2]; # "inner"
print $array[3]; # "list"
print $array[4]; # "several"
print $array[5]; # "entries"
print $array[6]; # "cherries"
```

The same is true whether the fat comma is used or not:

用不用"=>"都还是一样的:

```perl
my %hash = (
        "beer" => "good",
        "bananas" => (
                "green"  => "wait",
                "yellow" => "eat",
        ),
);
# The above raises a warning because the hash was declared using a 7-element list

print $hash{"beer"};    # "good"
print $hash{"bananas"}; # "green"
print $hash{"wait"};    # "yellow";
print $hash{"eat"};     # undef, so prints "" and raises a warning
```

Of course, this does make it easy to concatenate multiple arrays together:

这样也更便于数组的连接:

```perl
my @bones   = ("humerus", ("jaw", "skull"), "tibia");
my @fingers = ("thumb", "index", "middle", "ring", "little");
my @parts   = (@bones, @fingers, ("foot", "toes"), "eyeball", "knuckle");
print @parts;
```

More on this shortly.

多的就不再详述了.

# Context 格式

Perl's most distinctive feature is that its code is context-sensitive. Every expression in Perl is evaluated either in scalar context or list context, depending on whether it is expected to produce a scalar or a list. Many Perl expressions and built-in functions display radically different behaviour depending on the context in which they are evaluated.

A scalar assignment such as $scalar = evaluates its expression in scalar context. In this case, the expression is "Mendeleev" and the returned value is the same scalar value "Mendeleev":

Perl最独特的特性是格式相关的. 每个Perl表达式都用标量格式或列表格式来求值, 这取决于需要生成标量还是列表. 很多Perl表达式和内置函数在对不同的的格式求值时表现出完全不一样的特性.

	my $scalar = "Mendeleev";

An array or hash assignment such as @array = or %hash = evaluates its expression in list context. A list value evaluated in list context returns the list, which then gets fed in to populate the array or hash:

像是"@array = "或是"%hash = "这样的数组和哈希表赋值语句是按列表格式求值的.

	my @array = ("Alpha", "Beta", "Gamma", "Pie");
	my %hash = ("Alpha" => "Beta", "Gamma" => "Pie");

No surprises so far.

A scalar expression evaluated in list context turns into a single-element list:

到这里还没什么好奇怪的. 列表格式用标量表达式求值会出来一个单成员的列表:

	my @array = "Mendeleev"; # same as 'my @array = ("Mendeleev");'

A list expression evaluated in scalar context returns the final scalar in the list:

标量格式用列表格式求值出来的是列表中的最后一个标量:

	my $scalar = ("Alpha", "Beta", "Gamma", "Pie"); # Value of $scalar is now "Pie"

An array expression (an array is different from a list, remember?) evaluated in scalar context returns the length of the array:

标量格式用数组表达式求值会出来数组的长度:

	my @array = ("Alpha", "Beta", "Gamma", "Pie");
	my $scalar = @array; # Value of $scalar is now 4

The print built-in function evaluates all of its arguments in list context. In fact, print accepts an unlimited list of arguments and prints each one after the other, which means it can be used to print arrays directly:

内置的`print`函数对所有的参数进行列表格式的求值. 实际上, `print`函数可以接收并依次打印出无限个参数, 这就可以用来直接打印出数组来:

	my @array = ("Alpha", "Beta", "Goo");
	my $scalar = "-X-";
	print @array;              # "AlphaBetaGoo";
	print $scalar, @array, 98; # "-X-AlphaBetaGoo98";

You can force any expression to be evaluated in scalar context using the scalar built-in function. In fact, this is why we use scalar to retrieve the length of an array.

You are not bound by law or syntax to return a scalar value when a subroutine is evaluated in scalar context, nor to return a list value in list context. As seen above, Perl is perfectly capable of fudging the result for you.

你可以用内置的`scalar`函数来强制对表达式进行标量格式的求值. 其实这也是为啥我们用`scalar`来获取数组的长度.

你的子程序可以用标量格式求值并返回一个标量值或者用列表格式求值并返回列表值, 这个是没有规则和语法限制的. 由此可见, Perl是很擅长迷惑人的.

# References and nested data structures 引用和嵌套数据结构

In the same way that lists cannot contain lists as elements, arrays and hashes cannot contain other arrays and hashes as elements. They can only contain scalars. Watch what happens when we try:

列表不能包含列表, 数组和哈希表也一样不能嵌套包含自身. 它们只能包含标量. 来看看这个会发生什么:

	my @outer = ("Sun", "Mercury", "Venus", undef, "Mars");
	my @inner = ("Earth", "Moon");
	$outer[3] = @inner;
	print $outer[3]; # "2"

`$outer[3]` is a scalar, so it demands a scalar value. When you try to assign an array value like @inner to it, @inner is evaluated in scalar context. This is the same as assigning scalar @inner, which is the length of array @inner, which is 2.

However, a scalar variable may contain a reference to any variable, including an array variable or a hash variable. This is how more complicated data structures are created in Perl.

A reference is created using a backslash.

`$outer[3]`是标量, 所以这个需要标量的值. 当你用数组`@inner`给它赋值的时候, `@inner`会被求值成标量格式. 就是数组的长度, 2.

然而, 标量是可以包含一个其他变量(包括哈希, 数组)的引用的. 这就是Perl里面创建更复杂的数据结构的手段.

引用用反斜杠创建:

	my $colour    = "Indigo";
	my $scalarRef = \$colour;

Any time you would use the name of a variable, you can instead just put some braces in, and, within the braces, put a reference to a variable instead.

当要用变量名的时候, 你可以用花括号里面的引用来代替:

	print $colour;         # "Indigo"
	print $scalarRef;      # e.g. "SCALAR(0x182c180)"
	print ${ $scalarRef }; # "Indigo"

As long as the result is not ambiguous, you can omit the braces too:

如果结果是明确的, 你也可以略去花括号:

	print $$scalarRef; # "Indigo"

If your reference is a reference to an array or hash variable, you can get data out of it using braces or using the more popular arrow operator, ->:

如果是一个数组或者哈希表的引用, 你可以用花括号或者`->`箭头操作符来取出数据.

	my @colours = ("Red", "Orange", "Yellow", "Green", "Blue");
	my $arrayRef = \@colours;
	print $colours[0];       # direct array access
	print ${ $arrayRef }[0]; # use the reference to get to the array
	print $arrayRef->[0];    # exactly the same thing
	my %atomicWeights = ("Hydrogen" => 1.008, "Helium" => 4.003, "Manganese" => 54.94);
	my $hashRef = \%atomicWeights;
	print $atomicWeights{"Helium"}; # direct hash access
	print ${ $hashRef }{"Helium"};  # use a reference to get to the hash
	print $hashRef->{"Helium"};     # exactly the same thing - this is very common

## Declaring a data structure 定义数据结构

Here are four examples, but in practice the last one is the most useful.

这里用四个例子, 但实际上最后一个是最有用的.

```perl
my %owner1 = (
	"name" => "Santa Claus",
	"DOB"  => "1882-12-25",
);
my $owner1Ref = \%owner1;

my %owner2 = (
	"name" => "Mickey Mouse",
	"DOB"  => "1928-11-18",
);
my $owner2Ref = \%owner2;

my @owners = ( $owner1Ref, $owner2Ref );

my $ownersRef = \@owners;

my %account = (
	"number" => "12345678",
	"opened" => "2000-01-01",
	"owners" => $ownersRef,
);
```

That's obviously unnecessarily laborious, because you can shorten it to:

这太费事了, 你可以简化为:

```perl
my %owner1 = (
	"name" => "Santa Claus",
	"DOB"  => "1882-12-25",
);

my %owner2 = (
	"name" => "Mickey Mouse",
	"DOB"  => "1928-11-18",
);

my @owners = ( \%owner1, \%owner2 );

my %account = (
	"number" => "12345678",
	"opened" => "2000-01-01",
	"owners" => \@owners,
);
```

It is also possible to declare anonymous arrays and hashes using different symbols. Use square brackets for an anonymous array and braces for an anonymous hash. The value returned in each case is a reference to the anonymous data structure in question. Watch carefully, this results in exactly the same %account as above:

也可以用不同的符号定义匿名数组和哈希表, 方括号对应数组, 花括号对应哈希表. 返回值分别是匿名数据结构的引用. 小心看了, 这里出来的跟上面的`%account`完全一样:

```perl
# Braces denote an anonymous hash
my $owner1Ref = {
	"name" => "Santa Claus",
	"DOB"  => "1882-12-25",
};

my $owner2Ref = {
	"name" => "Mickey Mouse",
	"DOB"  => "1928-11-18",
};

# Square brackets denote an anonymous array
my $ownersRef = [ $owner1Ref, $owner2Ref ];

my %account = (
	"number" => "12345678",
	"opened" => "2000-01-01",
	"owners" => $ownersRef,
);
```

Or, for short (and this is the form you should actually use when declaring complex data structures in-line):

或者, 简单一点的(这会是你实际用来声明内联的复杂数据结构的形式):

```perl
my %account = (
	"number" => "31415926",
	"opened" => "3000-01-01",
	"owners" => [
		{
			"name" => "Philip Fry",
			"DOB"  => "1974-08-06",
		},
		{
			"name" => "Hubert Farnsworth",
			"DOB"  => "2841-04-09",
		},
	],
);
```

## Getting information out of a data structure 从数据结构里取出信息

Now, let's assume that you still have %account kicking around but everything else (if there was anything else) has fallen out of scope. You can print the information out by reversing the same procedure in each case. Again, here are four examples, of which the last is the most useful:

如果你现在还在整这个`%account`, 你可以在每个例子里面倒转步骤来打印出信息. 这里又是四个例子, 依然是最后一个才最有用:

```perl
my $ownersRef = $account{"owners"};
my @owners    = @{ $ownersRef };
my $owner1Ref = $owners[0];
my %owner1    = %{ $owner1Ref };
my $owner2Ref = $owners[1];
my %owner2    = %{ $owner2Ref };
print "Account #", $account{"number"}, "\n";
print "Opened on ", $account{"opened"}, "\n";
print "Joint owners:\n";
print "\t", $owner1{"name"}, " (born ", $owner1{"DOB"}, ")\n";
print "\t", $owner2{"name"}, " (born ", $owner2{"DOB"}, ")\n";
```
Or, for short:

短点的:

```perl
my @owners = @{ $account{"owners"} };
my %owner1 = %{ $owners[0] };
my %owner2 = %{ $owners[1] };
print "Account #", $account{"number"}, "\n";
print "Opened on ", $account{"opened"}, "\n";
print "Joint owners:\n";
print "\t", $owner1{"name"}, " (born ", $owner1{"DOB"}, ")\n";
print "\t", $owner2{"name"}, " (born ", $owner2{"DOB"}, ")\n";
```

Or using references and the -> operator:

用引用和`->`的:

```perl
my $ownersRef = $account{"owners"};
my $owner1Ref = $ownersRef->[0];
my $owner2Ref = $ownersRef->[1];
print "Account #", $account{"number"}, "\n";
print "Opened on ", $account{"opened"}, "\n";
print "Joint owners:\n";
print "\t", $owner1Ref->{"name"}, " (born ", $owner1Ref->{"DOB"}, ")\n";
print "\t", $owner2Ref->{"name"}, " (born ", $owner2Ref->{"DOB"}, ")\n";
```

And if we completely skip all the intermediate values:

如果我们要完全跳过中间值:

```perl
print "Account #", $account{"number"}, "\n";
print "Opened on ", $account{"opened"}, "\n";
print "Joint owners:\n";
print "\t", $account{"owners"}->[0]->{"name"}, " (born ", $account{"owners"}->[0]->{"DOB"}, ")\n";
print "\t", $account{"owners"}->[1]->{"name"}, " (born ", $account{"owners"}->[1]->{"DOB"}, ")\n";
```

## How to shoot yourself in the foot with array references

This array has five elements:

一个5个元素的数组:

	my @array1 = (1, 2, 3, 4, 5);
	print @array1; # "12345"

This array, however, has ONE element (which happens to be a reference to an anonymous, five-element array):

而这个数组只有1个成员(5成员数组的匿名引用):

	my @array2 = [1, 2, 3, 4, 5];
	print @array2; # e.g. "ARRAY(0x182c180)"

This scalar is a reference to an anonymous, five-element array:

这个标量是5成员数组的匿名引用:

	my $array3Ref = [1, 2, 3, 4, 5];
	print $array3Ref;      # e.g. "ARRAY(0x22710c0)"
	print @{ $array3Ref }; # "12345"
	print @$array3Ref;     # "12345"

# Conditionals 条件

## if ... elsif ... else ...

No surprises here, other than the spelling of elsif:

除了`elsif`, 不要奇怪:

```perl
my $word = "antidisestablishmentarianism";
my $strlen = length $word;

if($strlen >= 15) {
        print "'", $word, "' is a very long word";
} elsif(10 <= $strlen && $strlen < 15) {
        print "'", $word, "' is a medium-length word";
} else {
        print "'", $word, "' is a a short word";
}
```

Perl provides a shorter "statement if condition" syntax which is highly recommended for short statements:

Perl有一个更短的"[statement] if [condition]"语法, 更适用于缩短语句:

	print "'", $word, "' is actually enormous" if $strlen >= 20;

## unless ... else ...

```perl
my $temperature = 20;

unless($temperature > 30) {
        print $temperature, " degrees Celsius is not very hot";
} else {
        print $temperature, " degrees Celsius is actually pretty hot";
}
```

unless blocks are generally best avoided like the plague because they are very confusing. An "unless [... else]" block can be trivially refactored into an "if [... else]" block by negating the condition [or by keeping the condition and swapping the blocks]. Mercifully, there is no elsunless keyword.

This, by comparison, is highly recommended because it is so easy to read:

`unless`块很容易弄混, 最好像躲瘟疫一样避开. `unless [...else]`块可以通过对条件取反(或者条件不变交换语句块)来容易的变换成`if [...else]`块. 而且, `elsunless`关键字是没有的.

但是下面这句相比之下是比较推荐的, 因为读起来更简单:

	print "Oh no it's too cold" unless $temperature > 15;

## Ternary operator 条件运算符

The ternary operator ?: allows simple if statements to be embedded in a statement. The canonical use for this is singular/plural forms:

条件运算符`?:`可以把if表达式嵌入到语句里. 典型的用法是下面这个单数/复数形式:

	my $gain = 48;
	print "You gained ", $gain, " ", ($gain == 1 ? "experience point" : "experience points"), "!";

Aside: singulars and plurals are best spelled out in full in both cases. Don't do something clever like the following, because anybody searching the codebase to replace the words "tooth" or "teeth" will never find this line:

并且, 单数和复数项最好都拼全. 像是下面这样耍小聪明的就不要了, 因为在下面代码里面要搜索替换"tooth"或"teeth"是找不到这句的:

	my $lost = 1;
	print "You lost ", $lost, " t", ($lost == 1 ? "oo" : "ee"), "th!";

Ternary operators may be nested:

条件运算符是可以嵌套的

```perl
my $eggs = 5;
print "You have ", $eggs == 0 ? "no eggs" :
                   $eggs == 1 ? "an egg"  :
                   "some eggs";
```

if statements evaluate their conditions in scalar context. For example, if(@array) returns true if and only if @array has 1 or more elements. It doesn't matter what those elements are - they may contain undef or other false values for all we care.

if表达式对条件判断是用标量格式求值的. 比如`if(@array)`, 数组非空的话就返回`true`, 里面有些什么元素(如我们所关心的`undef`或其他`false`值)是不打紧的.

# Loops 循环

There's More Than One Way To Do It.

Perl has a conventional while loop:

有很多的做法, Perl有一个很便捷的`while`循环:

```perl
my $i = 0;
while($i < scalar @array) {
        print $i, ": ", $array[$i];
        $i++;
}
```

Perl also offers the until keyword:

也提供了`until`关键字:

```perl
my $i = 0;
until($i >= scalar @array) {
        print $i, ": ", $array[$i];
        $i++;
}
```

These do loops are almost equivalent to the above (a warning would be raised if @array were empty):

`do-while`循环跟上面的基本一致(`@array`为空则会有警告):

```perl
my $i = 0;
do {
        print $i, ": ", $array[$i];
        $i++;
} while ($i < scalar @array);
```

and

还有`do-until`,

```perl
my $i = 0;
do {
        print $i, ": ", $array[$i];
        $i++;
} until ($i >= scalar @array);
```

Basic C-style for loops are available too. Notice how we put a my inside the for statement, declaring $i only for the scope of the loop:

C风格的`for循环`也是可以用的. 要注意我们放了`my`赋值语句进去, 声明的`$i`的作用域只在这个循环体内:

```perl
for(my $i = 0; $i < scalar @array; $i++) {
        print $i, ": ", $array[$i];
}
# $i has ceased to exist here, which is much tidier.
```

This kind of for loop is considered old-fashioned and should be avoided where possible. Native iteration over a list is much nicer. Note: unlike PHP, the for and foreach keywords are synonyms. Just use whatever looks most readable:

下面这种循环是老式的, 尽量少用.

```perl
foreach my $string ( @array ) {
        print $string;
}
```

If you do need the indices, the range operator .. creates an anonymous list of integers:

如果你需要累加数, 范围操作符`..`会创建一个匿名整数表.

```perl
foreach my $i ( 0 .. $#array ) {
        print $i, ": ", $array[$i];
}
```

You can't iterate over a hash. However, you can iterate over its keys. Use the keys built-in function to retrieve an array containing all the keys of a hash. Then use the foreach approach that we used for arrays:

哈希表遍历不来. 不过, 你可以遍历键值. 用内建的`keys`函数可以获取哈希表的键值数组:

```perl
foreach my $key (keys %scientists) {
        print $key, ": ", $scientists{$key};
}
```

Since a hash has no underlying order, the keys may be returned in any order. Use the sort built-in function to sort the array of keys alphabetically beforehand:

由于哈希表的键值不是默认顺序的, 键值会以任意顺序返回. 可以用内置的`sort`函数预先对键值数组进行字母排序:

```perl
foreach my $key (sort keys %scientists) {
        print $key, ": ", $scientists{$key};
}
```

If you don't provide an explicit iterator, Perl uses a default iterator, $_. $_ is the first and friendliest of the built-in variables:

如果你没有提供显式的迭代器, Perl会用默认的迭代器`$_`. `$_`第一个也是最友好的内置变量:

```perl
foreach ( @array ) {
        print $_;
}
```

If using the default iterator, and you only wish to put a single statement inside your loop, you can use the super-short loop syntax:

如果你用了默认迭代器, 而且你只想在循环体里面写一个语句, 你就可以用这个超短的循环语法:

```perl
print $_ foreach @array;
```

## Loop control

next and last can be used to control the progress of a loop. In most programming languages these are known as continue and break respectively. We can also optionally provide a label for any loop. By convention, labels are written in ALLCAPITALS. Having labelled the loop, next and last may target that label. This example finds primes below 100:

下一条也是最后一条可以用来控制循环过程. 在大多编程语言里面这些就是`continue`和`break`. 我们也可以选择性的为循环体设标号. 按照惯例, 标号要用大写. 标记了循环体之后就可以指定这个循环体了. 下面的例子是找出100以内的质数:

```perl
CANDIDATE: for my $candidate ( 2 .. 100 ) {
        for my $divisor ( 2 .. sqrt $candidate ) {
                next CANDIDATE if $candidate % $divisor == 0;
        }
        print $candidate." is prime\n";
}
```

# Array functions 数组函数

## In-place array modification

We'll use @stack to demonstrate these:

我们用`@stack`来表示这些:

```perl
my @stack = ("Fred", "Eileen", "Denise", "Charlie");
print @stack; # "FredEileenDeniseCharlie"
```

pop extracts and returns the final element of the array. This can be thought of as the top of the stack:

`pop`抽出并返回最后一个数组元素. 这个可以类比成栈头:

```perl
print pop @stack; # "Charlie"
print @stack;     # "FredEileenDenise"
```

push appends extra elements to the end of the array:

`push`在数组的尾部追加一个元素:

```perl
push @stack, "Bob", "Alice";
print @stack; # "FredEileenDeniseBobAlice"
```

shift extracts and returns the first element of the array:

`shift`抽出并返回数组的第一个元素:

```perl
print shift @stack; # "Fred"
print @stack;       # "EileenDeniseBobAlice"
```

unshift inserts new elements at the beginning of the array:

`unshift`在数组的第一个元素前插入新元素:

```perl
unshift @stack, "Hank", "Grace";
print @stack; # "HankGraceEileenDeniseBobAlice"
```

pop, push, shift and unshift are all special cases of splice. splice removes and returns an array slice, replacing it with a different array slice:

`pop, push, shift, unshift`都是`splice`的特性情形. `splice`可以删除数组并返回一个数据段, 是用一个不同的数组段来替换的:

```perl
print splice(@stack, 1, 4, "<<<", ">>>"); # "GraceEileenDeniseBob"
print @stack;                             # "Hank<<<>>>Alice"
```

## Creating new arrays from old

Perl provides the following functions which act on arrays to create other arrays.

The join function concatenates many strings into one:

Perl用下列的函数作用与数组并产生新数组.

`join`函数可以把多个字符串连成一个:

```perl
my @elements = ("Antimony", "Arsenic", "Aluminum", "Selenium");
print @elements;             # "AntimonyArsenicAluminumSelenium"
print "@elements";           # "Antimony Arsenic Aluminum Selenium"
print join(", ", @elements); # "Antimony, Arsenic, Aluminum, Selenium"
```

In list context, the reverse function returns a list in reverse order. In scalar context, reverse concatenates the whole list together and then reverses it as a single word.

列表格式下, `reverse`函数把列表项翻转. 标量格式下, `reverse`把所有的列表项连到一起然后把整个字符串翻转.

```perl
print reverse("Hello", "World");        # "WorldHello"
print reverse("HelloWorld");            # "HelloWorld"
print scalar reverse("HelloWorld");     # "dlroWolleH"
print scalar reverse("Hello", "World"); # "dlroWolleH"
```

The map function takes an array as input and applies an operation to every scalar $_ in this array. It then constructs a new array out of the results. The operation to perform is provided in the form of a single expression inside braces:

`map`函数输入一个数组并对数组里每个标量`$_`施用同一个操作. 然后这样就建立了一个新的数组. 要执行的操作是花括号里的一个表达式.

```perl
my @capitals = ("Baton Rouge", "Indianapolis", "Columbus", "Montgomery", "Helena", "Denver", "Boise");

print join ", ", map { uc $_ } @capitals;
# "BATON ROUGE, INDIANAPOLIS, COLUMBUS, MONTGOMERY, HELENA, DENVER, BOISE"
```

The grep function takes an array as input and returns a filtered array as output. The syntax is similar to map. This time, the second argument is evaluated for each scalar $_ in the input array. If a boolean true value is returned, the scalar is put into the output array, otherwise not.

`grep`函数输入一个数组并返回过滤的数组. 语法跟`map`类似. 这次, 第二个参数是对输入数组的每个标量`$_`求值. 如果布尔表达式为真, 那么这个标量就会被放到输出数组里, 反之则不会.

```perl
print join ", ", grep { length $_ == 6 } @capitals;
# "Helena, Denver"
```

Obviously, the length of the resulting array is the number of successful matches, which means you can use grep to quickly check whether an array contains an element:

明显的, 输出数组的元素长度就是跟前面的数字成功匹配的, 这意味着你可以用`grep`快速的查找数组里是否包含某个元素:

	print scalar grep { $_ eq "Columbus" } @capitals; # "1"

grep and map may be combined to form list comprehensions, an exceptionally powerful feature conspicuously absent from many other programming languages.

By default, the sort function returns the input array, sorted into lexical (alphabetical) order:

`grep`和`map`可以组合起来用于列表识别, 这个是很多其他编程语言所不具有的很强的特性.

`sor`函数默认的根据输入数组输出按语义(字母)排序的数组: 

```perl
my @elevations = (19, 1, 2, 100, 3, 98, 100, 1056);

print join ", ", sort @elevations;
# "1, 100, 100, 1056, 19, 2, 3, 98"
```

However, similar to grep and map, you may supply some code of your own. Sorting is always performed using a series of comparisons between two elements. Your block receives $a and $b as inputs and should return -1 if $a is "less than" $b, 0 if they are "equal" or 1 if $a is "greater than" $b.

The cmp operator does exactly this for strings:

然而, 跟`grep`和`map`一样, 你可以施加一些你自己的代码. 排序总是按照一组元素间的比较规则来进行的. 像是你的模块的输入是`$a`和`$b`, `$a`小于`$b`的话就要输出-1, 相等的话输出0, 大于的话输出1. `cmp`就是作用于字符串的这个功能的操作符:

```perl
print join ", ", sort { $a cmp $b } @elevations;
# "1, 100, 100, 1056, 19, 2, 3, 98"
```

The "spaceship operator", <=>, does the same for numbers:

飞船操作符`<=>`是作用于数字的这个功能的操作符:

```perl
print join ", ", sort { $a <=> $b } @elevations;
# "1, 2, 3, 19, 98, 100, 100, 1056"
```

$a and $b are always scalars, but they can be references to quite complex objects which are difficult to compare. If you need more space for the comparison, you can create a separate subroutine and provide its name instead:

`$a`和`$b`都是标量, 但是它们都可以是很复杂难以用来比较的对象的引用. 如果你对比较有更多的需求, 可以自己起名另外写一个子程序:

```perl
sub comparator {
        # lots of code...
        # return -1, 0 or 1
}

print join ", ", sort comparator @elevations;
```

You can't do this for grep or map operations.

Notice how the subroutine and block are never explicitly provided with $a and $b. Like $_, $a and $b are, in fact, global variables which are populated with a pair of values to be compared each time.

这个操作对`map`和`grep`是没用的.

注意子程序和块里面里面都是没有显示的用`$a`和`$b`的. 实际上, 像`$_`, `$a`, `$b`这些都是用来作对比的全局变量. 

# Built-in functions 内置函数

By now you have seen at least a dozen built-in functions: print, sort, map, grep, keys, scalar and so on. Built-in functions are one of Perl's greatest strengths. They

目前为止, 你已经见过了很多的内置函数: print, sort, map, grep, keys, scalar等. 内置函数是Perl的强点, 它们

- are numerous
很多
- are very useful
很有用
- are extensively documented
有大量的文档
- vary greatly in syntax, so check the documentation
语法上区别很大, 所以要多翻阅文档
- sometimes accept regular expressions as arguments
有时需要正则表达式做输入
- sometimes accept entire blocks of code as arguments
有时需要整个代码块做输入
- sometimes don't require commas between arguments
有时参数间不用加逗号
- sometimes will consume an arbitrary number of comma-separated arguments and sometimes will not
有时会需要很多的逗号分隔的参数, 有时不需要
- sometimes will fill in their own arguments if too few are supplied
参数太少的时候有时会自己填充参数
- generally don't require brackets around their arguments except in ambiguous circumstances
大多数时候参数是不用加花括号的, 除了语义模糊的情况

The best advice regarding built-in functions is to know that they exist. Skim the documentation for future reference. If you are carrying out a task which feels like it's low-level and common enough that it's been done many times before, the chances are that it has.

关于内置函数的最好建议就是去了解它们的存在. 在文档上瞄一眼特性参考. 如果你认为你要做的任务是底层, 普通到之前完成过好几次的, 它们就可能已经在了.

# User-defined subroutines 自定义子函数

Subroutines are declared using the sub keyword. In contrast with built-in functions, user-defined subroutines always accept the same input: a list of scalars. That list may of course have a single element, or be empty. A single scalar is taken as a list with a single element. A hash with N elements is taken as a list with 2N elements.

Although the brackets are optional, subroutines should always be invoked using brackets, even when called with no arguments. This makes it clear that a subroutine call is happening.

Once you're inside a subroutine, the arguments are available using the built-in array variable @_. Example:

子函数是用`sub`关键字定义的. 跟内置函数不同的是, 自定义函数总是接收同样的输入: 列表或标量. 列表当然可能只有一个元素或是空列表. 标量被认为是只有一个元素的列表. N个元素的哈希表被认为是2N个元素的列表.

花括号是选用的, 但是子函数应该都用上, 即使是在没有参数的情况下调用. 这个就使得调用子函数这个事实变得很明显了.

```perl
sub hyphenate {

  # Extract the first argument from the array, ignore everything else
  my $word = shift @_;

  # An overly clever list comprehension
  $word = join "-", map { substr $word, $_, 1 } (0 .. (length $word) - 1);
  return $word;
}

print hyphenate("exterminate"); # "e-x-t-e-r-m-i-n-a-t-e"
```

# Perl calls by reference Perl引用调用

Unlike almost every other major programming language, Perl calls by reference. This means that the variables or values available inside the body of a subroutine are not copies of the originals. They are the originals.

跟其他大多数主要的编程语言对不一样的是, Perl是通过引用来调用的. 这个意味着子函数体里的变量和值都是原数据而不是源数据的拷贝.

```perl
my $x = 7;

sub reassign {
  $_[0] = 42;
}

reassign($x);
print $x; # "42"
```

If you try something like

如果你试一下下面这种:

	reassign(8);

then an error occurs and execution halts, because the first line of reassign() is equivalent to

这里就会停止执行并报错, 因为`reassign()`的第一行成了:

	8 = 7;

which is obviously nonsense.

The lesson to learn is that in the body of a subroutine, you should always unpack your arguments before working with them.

这显然是不合理的. 这个的教训就是在子函数的函数体里, 在使用参数之前要先"解压".

## Unpacking arguments 解压参数

There's More Than One Way To unpack @_, but some are superior to others.

The example subroutine left_pad below pads a string out to the required length using the supplied pad character. (The x function concatenates multiple copies of the same string in a row.) (Note: for brevity, these subroutines all lack some elementary error checking, i.e. ensuring the pad character is only 1 character, checking that the width is greater than or equal to the length of existing string, checking that all needed arguments were passed at all.)

left_pad is typically invoked as follows:

有不止一种方法可以用来解压`@_`, 但是有一些是比较好的.

下面的示例函数`left_pad`用指定的字符来向左填充原字符串直至长度到达指定的数值. (x函数把多个同样的字符串连接成一串.)(注: 为简单起见, 这些子函数都缺乏元错误检查)

	print left_pad("hello", 10, "+"); # "+++++hello"

- Unpacking @_ entry by entry is effective but not terribly pretty:
对`@_`的每个成员做解压是有效的, 但是不优雅:

```perl
sub left_pad {
        my $oldString = $_[0];
        my $width     = $_[1];
        my $padChar   = $_[2];
        my $newString = ($padChar x ($width - length $oldString)) . $oldString;
        return $newString;
}
```

- Unpacking @_ by removing data from it using shift is recommended for up to 4 arguments:
4个以内的参数用`shift`来移出`@_`是推荐的做法:

```perl
sub left_pad {
        my $oldString = shift @_;
        my $width     = shift @_;
        my $padChar   = shift @_;
        my $newString = ($padChar x ($width - length $oldString)) . $oldString;
        return $newString;
}
```

If no array is provided to the shift function, then it operates on @_ implicitly. This approach is seen very commonly:

如果`shift`之后不带数组参数, 那`shift`就隐式的作用于`@_`. 这种方法是很常见的:

```perl
sub left_pad {
        my $oldString = shift;
        my $width     = shift;
        my $padChar   = shift;
        my $newString = ($padChar x ($width - length $oldString)) . $oldString;
        return $newString;
}
```

Beyond 4 arguments it becomes hard to keep track of what is being assigned where.

4个以上的参数的话就要追踪哪个变量幅值了哪个参数就有难度了.

- You can unpack @_ all in one go using multiple simultaneous scalar assignment. Again, this is okay for up to 4 arguments:
可以在一个语句里面对多个标量进行`@_`的同时赋值, 这也是适用于4个以下参数:

```perl
sub left_pad {
        my ($oldString, $width, $padChar) = @_;
        my $newString = ($padChar x ($width - length $oldString)) . $oldString;
        return $newString;
}
```

- For subroutines with large numbers of arguments or where some arguments are optional or cannot be used in combination with others, best practice is to require the user to provide a hash of arguments when calling the subroutine, and then unpack @_ back into that hash of arguments. For this approach, our subroutine call would look a little different:
对有大量参数或者是可变参数或者是带有不能与"其他参数结合的"参数的子函数, 最好的方法是要求使用者在调用函数事提供参数哈希表, 然后把`@_`解压成哈希参数. 这种方法的话, 函数调用看起来就有点不同了:

```perl
print left_pad("oldString" => "pod", "width" => 10, "padChar" => "+");
```

And the subroutine itself looks like this:

然后子函数看起来就是像这样:

```perl
sub left_pad {
        my %args = @_;
        my $newString = ($args{"padChar"} x ($args{"width"} - length $args{"oldString"})) . $args{"oldString"};
        return $newString;
}
```

## Returning values

Like other Perl expressions, subroutine calls may display contextual behaviour. You can use the wantarray function (which should be called wantlist but never mind) to detect what context the subroutine is being evaluated in, and return a result appropriate to that context:

跟其他Perl表达式一样, 子函数调用会有格式行为. 你可以用`wantarry`函数(其实应该是`wantlist`函数但是不管了)来检测一下子函数用的是用哪种格式求值的, 然后可以用来输出合适的格式:

```perl
sub contextualSubroutine {
        # Caller wants a list. Return a list
        return ("Everest", "K2", "Etna") if wantarray;

        # Caller wants a scalar. Return a scalar
        return 3;
}

my @array = contextualSubroutine();
print @array; # "EverestK2Etna"

my $scalar = contextualSubroutine();
print $scalar; # "3"
```

# System calls

Apologies if you already know the following non-Perl-related facts. Every time a process finishes on a Windows or Linux system (and, I assume, on most other systems), it concludes with a 16-bit status word. The highest 8 bits constitute a return code between 0 and 255 inclusive, with 0 conventionally representing unqualified success, and other values representing various degrees of failure. The other 8 bits are less frequently examined - they "reflect mode of failure, like signal death and core dump information".

You can exit from a Perl script with the return code of your choice (from 0 to 255) using exit.

Perl provides More Than One Way To - in a single call - spawn a child process, pause the current script until the child process has finished, and then resume interpretation of the current script. Whichever method is used, you will find that immediately afterwards, the built-in scalar variable $? has been populated with the status word that was returned from that child process's termination. You can get the return code by taking just the highest 8 of those 16 bits: $? >> 8.

The system function can be used to invoke another program with the arguments listed. The value returned by system is the same value with which $? is populated:

如果你已经知道了下面这些无关Perl的事实, 那要跟你道个歉. 在Windows或Linux系统(或者我猜测的大多数系统)中, 每次一个进程结束, 会归结成一个16位的状态位. 高8位组成了包括0~255的返回码(0表示成功, 其他只表示各种失败等级). 比较少用的另外8位指示了失败的模式, 像是信号死亡或者核心废弃信息. 

你可以用`exit`再选一个0~255的返回码来退出Perl脚本.

Perl有不止一种方式来创建子进程(单个调用), 暂停当前脚本至子进程完成, 然后继续解释执行当前脚本. 无论你用哪种方式, 

```perl
my $rc = system "perl", "anotherscript.pl", "foo", "bar", "baz";
$rc >>= 8;
print $rc; # "37"
```

Alternatively, you can use backticks `` to run an actual command at the command line and capture the standard output from that command. In scalar context the entire output is returned as a single string. In list context, the entire output is returned as an array of strings, each one representing a line of output.

此外, 你可以用反单引号对\`\`来运行命令行, 然后可以读取命令的标准输出. 标量格式下输出就会出单个字符串, 列表格式下就出每个成员一列的字符串数组.

```perl
my $text = `perl anotherscript.pl foo bar baz`;
print $text; # "foobarbaz"
```

This is the behaviour which would be seen if anotherscript.pl contained, for example:

这个就是在查看`anotherscript.pl`是不是有内容的时候会看到的现象, 比如:

```perl
use strict;
use warnings;

print @ARGV;
exit 37;
```

# Files and file handles 文件和文件句柄

A scalar variable may contain a file handle instead of a number/string/reference or undef. A file handle is essentially a reference to a specific location inside a specific file.

Use open to turn a scalar variable into a file handle. open must be supplied with a mode. The mode < indicates that we wish to open the file to read from it:

除数字, 字符串, 引用和`undef`外, 标量还可以存放文件句柄. 文件句柄实际上就是一个特定的文件内的指向一个特定路径的引用. 

用`open`可以把标量转成文件句柄. `open`必须要用带上模式使用, `<`模式指的是打开并读取:

```perl
my $f = "text.txt";
my $result = open my $fh, "<", $f;

if(!$result) {
        die "Couldn't open '".$f."' for reading because: ".$!;
}
```

If successful, open returns a true value. Otherwise, it returns false and an error message is stuffed into the built-in variable $!. As seen above, you should always check that the open operation completed successfully. This checking being rather tedious, a common idiom is:

如果成功了, `open`会返回一个`true`值. 不然的话, 会返回`false`, 同时内建的`$!`变量会填入错误信息. 由此, 每次`open`的时候都应该检查一下是不是成功执行了. 上面的检查比较烦人, 常用方法是:

	open(my $fh, "<", $f) || die "Couldn't open '".$f."' for reading because: ".$!;

Note the need for parentheses around the open call's arguments.

To read a line of text from a filehandle, use the readline built-in function. readline returns a full line of text, with a line break intact at the end of it (except possibly for the final line of the file), or undef if you've reached the end of the file.

注意`open`调用的时候要用括号括住参数.

从文件句柄里面读出一行文本可以用内置的`readline`函数. `readline`返回一个末尾带换行符的整行文本(除非是文件的最后一行), 已经在文件的最后的话就要返回undef了.

```perl
while(1) {
        my $line = readline $fh;
        last unless defined $line;
        # process the line...
}
```

To truncate that possible trailing line break, use chomp:

用`chomp`可以取掉尾部可能有的换行符:

	chomp $line;

Note that chomp acts on \$line in place. `$line = chomp $` line is probably not what you want.

You can also use eof to detect that the end of the file has been reached:

`chomp`在适当的位置作用于`$line`, 所以`$line = chomp $`可能不是你要的.

你也可以用`eof`来检测是不是到文件尾了:

```perl
while(!eof $fh) {
        my $line = readline $fh;
        # process $line...
}
```

But beware of just using while(my \$line = readline \$fh), because if \$line turns out to be "0", the loop will terminate early. If you want to write something like that, Perl provides the <> operator which wraps up readline in a fractionally safer way. This is very commonly-seen and perfectly safe:

要注意使用`while(my \$line = readline \$fh)`, 因为如果`$line`刚好是'0', 那while就会提早终止. 如你是要写点什么东西, Perl的`<>`操作符会更安全一点的处理好`readline`. 下面这个就是很常见又安全的的形式:

```perl
while(my $line = <$fh>) {
        # process $line...
}
```

And even:

或者是:

```perl
while(<$fh>) {
        # process $_...
}
```

Writing to a file involves first opening it in a different mode. The mode > indicates that we wish to open the file to write to it. (> will clobber the content of the target file if it already exists and has content. To merely append to an existing file, use mode >>.) Then, simply provide the filehandle as a zeroth argument for the print function.

写文件的第一步包括用各种模式打开这个文件. `>`模式指的是打开并写文件(`>`会删掉原文件的所有内容, 只是要在原文件后面追加内容的话要用`>>`模式). 然后, 可以简单的把文件句柄作为`print`函数的第一个参数.

```perl
open(my $fh2, ">", $f) || die "Couldn't open '".$f."' for writing because: ".$!;
print $fh2 "The eagles have left the nest";
```

Notice the absence of a comma between $fh2 and the next argument.

File handles are actually closed automatically when they drop out of scope, but otherwise:

注意`$fh2`跟后一个参数之间是没有逗号的.

文件句柄在作用域外是会自动关闭的, 然而可以用:

```perl
close $fh2;
close $fh;
```

Three filehandles exist as global constants: STDIN, STDOUT and STDERR. These are open automatically when the script starts. To read a single line of user input:

STDIN, STDOUT和STDERR这三个文件句柄是全局常数. 脚本开始时会自动打开. 要读出用户输入的一行:

```perl
my $line = <STDIN>;
```

To just wait for the user to hit Enter:

等待用户按回车:

```perl
<STDIN>;
```

Calling \<\> with no filehandle reads data from STDIN, or from any files named in arguments when the Perl script was called.

As you may have gathered, print prints to STDOUT by default if no filehandle is named.

调用没有文件句柄的空`<>`的时候是从STDIN读入的数据, 或是Perl脚本调用的时候的任意一个参数表中的文件.

到这里你可能已经知道了, 就是没有指定文件句柄的时候`print`默认输出到`STDOUT`.

## File tests

The function -e is a built-in function which tests whether the named file exists.

`-e`内建函数是用来判断指定的文件是否存在.

```perl
print "what" unless -e "/usr/bin/perl";
```

The function -d is a built-in function which tests whether the named file is a directory.

The function -f is a built-in function which tests whether the named file is a plain file.

These are just three of a large class of functions of the form -X where X is some lower- or upper-case letter. These functions are called file tests. Note the leading minus sign. In a Google query, the minus sign indicates to exclude results containing this search term. This makes file tests hard to Google for! Just search for "perl file test" instead.

`-d`内建函数是用来测试指定的文件是否是一个路径.

`-f`内建函数是用来测试指定的文件是否是一个文件.

这三个都归属于`-X`函数族(X是一些小写或大写字母). 这些函数叫做文件测试函数. 要注意前面的减号. 在Google搜索中, 减号表示在结果里面排除减号之后的文字. 这个就让文件测试变得难以搜索了. 那就用"Perl文件测试"来代替吧.

# Regular expressions 正则表达式

Regular expressions appear in many languages and tools other than Perl. Perl's core regular expression syntax is basically the same as everywhere else, but Perl's full regular expression capabilities are terrifyingly complex and difficult to understand. The best advice I can give you is to avoid this complexity wherever possible.

Match operations are performed using `=~ m//`. In scalar context, `=~ m//` returns true on success, false on failure.

正则表达式在很多Perl之外的语言和工具里都出现了. Perl的核心正则表达式语法跟其他的基本一样, 但是其完整的正则表达式内容是很复杂很难懂的. 我的建议是尽量少用这些复杂的东西.

匹配操作是用的`=~ m//`. 标量格式下, `=~ m//`匹配成功返回"true", 失败返回"false".

```perl
my $string = "Hello world";
if($string =~ m/(\w+)\s+(\w+)/) {
        print "success";
}
```

Parentheses perform sub-matches. After a successful match operation is performed, the sub-matches get stuffed into the built-in variables `$1, $2, $3, ...`:

括号里是子表达式匹配. 匹配操作成功的话, 子表达式会填入内置的变量`$1, $2, $3...`:

```perl
print $1; # "Hello"
print $2; # "world"
```

In list context, `=~ m//` returns `$1, $2, ...` as a list.

列表格式下`=~ m//`匹配输出`$1, $2,...`这样的一个列表.

```perl
my $string = "colourless green ideas sleep furiously";
my @matches = $string =~ m/(\w+)\s+((\w+)\s+(\w+))\s+(\w+)\s+(\w+)/;

print join ", ", map { "'".$_."'" } @matches;
# prints "'colourless', 'green ideas', 'green', 'ideas', 'sleep', 'furiously'"
```

Substitution operations are performed using `=~ s///`.

替换操作用`=~ s///`.

```perl
my $string = "Good morning world";
$string =~ s/world/Vietnam/;
print $string; # "Good morning Vietnam"
```

Notice how the contents of `$string` have changed. You have to pass a scalar variable on the left-hand side of an `=~ s///` operation. If you pass a literal string, you'll get an error.

The `/g` flag indicates "group match".

In scalar context, each `=~ m//g` call finds another match after the previous one, returning true on success, false on failure. You can access `$1` and so on afterwards in the usual way. For example:

要注意`$string`被替换了. 用`=~ s///`操作的左边要用标量, 如果你用文字字符串, 就会出来个错误.

`/g`标志表示"全组匹配". 标量格式中, 每次`=~ m//g`的调用查找下一个匹配, 成功返回`true`, 失败返回`false`. 你可以用普通的方式访问`$1`和后续的值, 比如:

```perl
my $string = "a tonne of feathers or a tonne of bricks";
while($string =~ m/(\w+)/g) {
  print "'".$1."'\n";
}
```

In list context, an `=~ m//g` call returns all of the matches at once.

列表格式下, `=~ m//g`返回所有的匹配.

```perl
my @matches = $string =~ m/(\w+)/g;
print join ", ", map { "'".$_."'" } @matches;
```

An `=~ s///g` call performs a global search/replace and returns the number of matches. Here, we replace all vowels with the letter "r".

`=~ s///g`执行全局的查找替换操作并返回匹配的个数. 这里, 我们用"r"来替换所有的元音.

```perl
# Try once without /g.
$string =~ s/[aeiou]/r/;
print $string; # "r tonne of feathers or a tonne of bricks"

# Once more.
$string =~ s/[aeiou]/r/;
print $string; # "r trnne of feathers or a tonne of bricks"

# And do all the rest using /g
$string =~ s/[aeiou]/r/g;
print $string, "\n"; # "r trnnr rf frrthrrs rr r trnnr rf brrcks"
```

The /i flag makes matches and substitutions case-insensitive.

The /x flag allows your regular expression to contain whitespace (e.g., line breaks) and comments.

`/i`标志表示查找和替换是大小写敏感的, `/x`标志允许你的正则表达式包含空白(像是换行符)和注释.

```perl
"Hello world" =~ m/
  (\w+) # one or more word characters
  [ ]   # single literal space, stored inside a character class
  world # literal "world"
/x;

# returns true
```

# Modules and packages 模块和包

In Perl, modules and packages are different things.

Perl里面, 模块和包是不同的. 

## Modules

A module is a `.pm` file that you can include in another Perl file (script or module). A module is a text file with exactly the same syntax as a `.pl` Perl script. An example module might be located at `C:\foo\bar\baz\Demo\StringUtils.pm` or `/foo/bar/baz/Demo/StringUtils.pm`, and read as follows:

模块就是一个你可以用来包含其他Perl文件的(脚本或模块)的`.pm`文件. 模块是跟`.pl`Perl脚本语法完全一样的文本文件. 可以在`C:\foo\bar\baz\Demo\StringUtils.pm`或`/foo/bar/baz/Demo/StringUtils.pm`里看到这样的一个例子, 读取出来是:

```perl
use strict;
use warnings;

sub zombify {
        my $word = shift @_;
        $word =~ s/[aeiou]/r/g;
        return $word;
}

return 1;
```

Because a module is executed from top to bottom when it is loaded, you need to return a true value at the end to show that it was loaded successfully.

So that the Perl interpreter can find them, directories containing Perl modules should be listed in your environment variable PERL5LIB before calling perl. List the root directory containing the modules, don't list the module directories or the modules themselves:

由于模块载入时是自顶向下执行的, 你要在最后返回一个`true`来表示载入成功了. 在调用Perl前, 包含Perl模块的路径要先加入环境变量`PERL5LIB`, 这样Perl解释器才能找到这些模块. 列出包含模块的根目录就可以了, 不需要列出模块目录和模块本身;

```perl
set PERL5LIB=C:\foo\bar\baz;%PERL5LIB%
```

or

或, 

```perl
export PERL5LIB=/foo/bar/baz:$PERL5LIB
```

Once the Perl module is created and perl knows where to look for it, you can use the require built-in function to search for and execute it during a Perl script. For example, calling require `Demo::StringUtils` causes the Perl interpreter to search each directory listed in `PERL5LIB` in turn, looking for a file called `Demo/StringUtils.pm`. After the module has been executed, the subroutines that were defined there suddenly become available to the main script. Our example script might be called `main.pl` and read as follows:

Perl模块被创建并且被找到之后, 你就可以用`require`内置函数在Perl脚本里搜索并执行之. 比如, 调用`require Demo::StringUtils`会让Perl解释器依次在`PERL5LIB`里面搜索一个叫`Demo/StringUtils.pm`的文件. 这个模块执行之后, 模块里定义的子函数就可以在主脚本里使用了. 我们的例子`main.pl`是下面这样的:

```perl
use strict;
use warnings;

require Demo::StringUtils;

print zombify("i want brains"); # "r wrnt brrrns"
```

Note the use of the double colon `::` as a directory separator.

Now a problem surfaces: if `main.pl` contains many require calls, and each of the modules so loaded contains more require calls, then it can become difficult to track down the original declaration of the zombify() subroutine. The solution to this problem is to use packages.

注意俩冒号`::`是路径分割符.

这里就出现了一个问题: 如果`main.pl`里面有很多的`require`调用, 然后每个模块里面又有多个`require`调用, 这样一来要查找`zombify()`的原声明就有难度了. 这个问题的解决方案是用包.

## Packages

A package is a namespace in which subroutines can be declared. Any subroutine you declare is implicitly declared within the current package. At the beginning of execution, you are in the main package, but you can switch package using the package built-in function:

包就是用来声明子函数的命名空间. 你的任意子函数都隐式的在当前包里声明. 执行开始之初, 你就是在main这个包里, 然而你可以用`package`这个内建函数来切换命名空间:

```perl
use strict;
use warnings;

sub subroutine {
        print "universe";
}

package Food::Potatoes;

# no collision:
sub subroutine {
        print "kingedward";
}
```

Note the use of the double colon `::` as a namespace separator.

Any time you call a subroutine, you implicitly call a subroutine which is inside the current package. Alternatively, you can explicitly provide a package. See what happens if we continue the above script:

注意这里的俩冒号是命名空间分隔符.

每次你调用一个子函数的时候, 你是隐式的调用当前包内的子函数. 此外, 你可以显示的用一个包. 来看看继续上面的脚本会怎么样:

```perl
subroutine();                 # "kingedward"
main::subroutine();           # "universe"
Food::Potatoes::subroutine(); # "kingedward"
```

So the logical solution to the problem described above is to modify `C:\foo\bar\baz\Demo\StringUtils.pm` or `/foo/bar/baz/Demo/StringUtils.pm` to read:

所以对上面的问题, 合理的方案是修改`C:\foo\bar\baz\Demo\StringUtils.pm` 或 `/foo/bar/baz/Demo/StringUtils.pm`里面的:

```perl
use strict;
use warnings;

package Demo::StringUtils;

sub zombify {
        my $word = shift @_;
        $word =~ s/[aeiou]/r/g;
        return $word;
}

return 1;
```

And modify `main.pl` to read:

同时修改`main.pl`:

```perl
use strict;
use warnings;

require Demo::StringUtils;

print Demo::StringUtils::zombify("i want brains"); # "r wrnt brrrns"
```

Now read this next bit carefully.

Packages and modules are two completely separate and distinct features of the Perl programming language. The fact that they both use the same double colon delimiter is a huge red herring. It is possible to switch packages multiple times over the course of a script or module, and it is possible to use the same package declaration in multiple locations in multiple files. Calling require Foo::Bar does not look for and load a file with a package Foo::Bar declaration somewhere inside it, nor does it necessarily load subroutines in the Foo::Bar namespace. Calling require Foo::Bar merely loads a file called Foo/Bar.pm, which need not have any kind of package declaration inside it at all, and in fact might declare package Baz::Qux and other nonsense inside it for all you know.

Likewise, a subroutine call `Baz::Qux::processThis()` need not necessarily have been declared inside a file named `Baz/Qux.pm`. It could have been declared literally anywhere.
Separating these two concepts is one of the stupidest features of Perl, and treating them as separate concepts invariably results in chaotic, maddening code. Fortunately for us, the majority of Perl programmers obey the following two laws:

现在来仔细的读读下面的文字.

包和模块是Perl里面俩全然不同的特性. 它们都用俩冒号做分隔符很容易引起误会. 一个模块或脚本里可以多次切换包, 同一个包也可以在多个文件的多处使用. 调用`require Foo::Bar`不是要搜索并载入一个里面定义了`Foo::Bar`的文件, 也不是载入`Foo::Bar`命名空间里的子函数. 调用`Foo::Bar`只是载入名为`Foo/Bar.pm`的一个文件, 这个文件不需要有任何的包定义, 而且实际上里面甚至可以是定义了`Baz::Qux`或其他毫不相干的包.

同样的, 一个`Baz::Qux::processThis()`的子函数是不用在`Baz/Qux.pm`这个文件里定义的. 它可能在任意的地方定义. 把这俩概念分开是Perl最傻的一个特性, 这一定会造成那种混乱的让人发疯的代码. 幸运的是, 大部分Perl程序员遵循着如下两条规则:

1. A Perl script (.pl file) must always contain exactly zero package declarations.
Perl脚本必须不包含包声明. 
2. A Perl module (.pm file) must always contain exactly one package declaration, corresponding exactly to its name and location. E.g. module `Demo/StringUtils.pm` must begin with package `Demo::StringUtils`.
Perl模块必须包含一个根据模块名和路径的包声明. 例如, `Demo/StringUtils.pm`必须以`Demo::StringUtils`开头.

Because of this, in practice you will find that most "packages" and "modules" produced by reliable third parties can be regarded and referred to interchangeably. However, it is important that you do not take this for granted, because one day you will meet code produced by a madman.

因此, 你会发现大多数第三方开发的包和模块是可以串着用的. 然而, 不能这么想当然, 因为你可能会碰到疯子写的代码.

# Object-oriented Perl 面向对象

Perl is not a great language for OO programming. Perl's OO capabilities were grafted on after the fact, and this shows.

Perl不是一个很好的面向对象的语言. Perl的OO特性来自以下的事实, 

- An object is simply a reference (i.e. a scalar variable) which happens to know which class its referent belongs to. To tell a reference that its referent belongs to a class, use bless. To find out what class a reference's referent belongs to (if any), use ref.
对象就是对所属类的引用(即标量). 用`bless`来指示一个类的引用. 用`ref`来引用是指向哪个类的(如果有的话).
- A method is simply a subroutine that expects an object (or, in the case of class methods, a package name) as its first argument. Object methods are invoked using `$obj->method()`; class methods are invoked using Package::Name->method().
方法简单的讲就是以对象(或用类方法的说法,包名)为第一个参数的子函数. 对象方法是用`$obj->method()`这个格式调用的, 类方法是用`Paceage::Name->method()`的格式调用的.
- A class is simply a package that happens to contain methods.
类简单的讲就是包含了方法的包.

A quick example makes this clearer. An example module `Animal.pm` containing a class Animal reads like this:

一个小例子来理一理. `Animal.pm`这个示例模块包含了一个这样的Animal类:

```perl
use strict;
use warnings;

package Animal;

sub eat {
        # First argument is always the object to act upon.
        my $self = shift @_;

        foreach my $food ( @_ ) {
                if($self->can_eat($food)) {
                        print "Eating ", $food;
                } else {
                        print "Can't eat ", $food;
                }
        }
}

# For the sake of argument, assume an Animal can eat anything.
sub can_eat {
        return 1;
}

return 1;
```

And we might make use of this class like so:

然后我们可以这样用:

```perl
require Animal;

my $animal = {
        "legs"   => 4,
        "colour" => "brown",
};                       # $animal is an ordinary hash reference
print ref $animal;       # "HASH"
bless $animal, "Animal"; # now it is an object of class "Animal"
print ref $animal;       # "Animal"
```

Note: literally any reference can be blessed into any class. It's up to you to ensure that (1) the referent can actually be used as an instance of this class and (2) that the class in question exists and has been loaded.

You can still work with the original hash in the usual way:

注: 说起来任何引用都是可以指向类的, 只要你能保证:

1. 这个引用是用来做类的实例.
2. 这个类是存在的而且已经载入了.

原来的哈希表还是可以像往常一样用:

```perl
print "Animal has ", $animal->{"legs"}, " leg(s)";
```

But you can now also call methods on the object using the same -> operator, like so:

这时你也可以用`->`操作符来调用对象的方法:

```perl
$animal->eat("insects", "curry", "eucalyptus");
```

This final call is equivalent to `Animal::eat($animal, "insects", "curry", "eucalyptus")`.

这个相当于`Animal::eat($animal, "insects", "curry", "eucalyptus")`.

## Constructors 构造函数

A constructor is a class method which returns a new object. If you want one, just declare one. You can use any name you like. For class methods, the first argument passed is not an object but a class name. In this case, "Animal":

构造函数就是返回新对象的类方法. 如果你需要, 定义个. 取啥名字都行. 对类方法来说, 第一个传入参数是类名而不是对象. 以这个例子来说就是"Animal":

```perl
use strict;
use warnings;

package Animal;

sub new {
        my $class = shift @_;
        return bless { "legs" => 4, "colour" => "brown" }, $class;
}

# ...etc.
```

And then use it like so:

然后可以这样用:

```perl
my $animal = Animal->new();
```

## Inheritance 继承

To create a class inheriting from a parent class, use use parent. Let's suppose we subclassed Animal with Koala, located at `Koala.pm`:

用`parent`可以创建一个继承父类的类. 假定我们在`Koala.pm`里新建`Koala`为`Animal`的子类:

```perl
use strict;
use warnings;

package Koala;

# Inherit from Animal
use parent ("Animal");

# Override one method
sub can_eat {
        my $self = shift @_; # Not used. You could just put "shift @_;" here
        my $food = shift @_;
        return $food eq "eucalyptus";
}

return 1;
```

And some sample code:

还有一些示例代码:

```perl
use strict;
use warnings;

require Koala;

my $koala = Koala->new();

$koala->eat("insects", "curry", "eucalyptus"); # eat only the eucalyptus
```

This final method call tries to invoke `Koala::eat($koala, "insects", "curry", "eucalyptus")`, but a subroutine eat() isn't defined in the Koala package. However, because Koala has a parent class Animal, the Perl interpreter tries calling `Animal::eat($koala, "insects", "curry", "eucalyptus")` instead, which works. Note how the class Animal was loaded automatically by Koala.pm.

Since use parent accepts a list of parent class names, Perl supports multiple inheritance, with all the benefits and horrors this entails.

最后的方法调用是想执行`Koala::eat($koala, "insects", "curry", "eucalyptus")`, 但是Koala包里面没有定义eat()子函数, 不过因为Koala有Animal父类, Perl解释器就会调用可用的`Animal::eat($koala, "insects", "curry", "eucalyptus")`. 要注意Animal类是通过`Koala.pm`自动载入的.

由于`parent`可以接收多个父类名, Perl是支持多重继承的, 有好有坏.

# BEGIN blocks

A BEGIN block is executed as soon as perl has finished parsing that block, even before it parses the rest of the file. It is ignored at execution time:

BEGIN块是Perl解析完之后就会马上执行的, 甚至是在解析玩文件的剩余部分之前. 这个块在执行的时候是被忽略的:

```perl
use strict;
use warnings;

print "This gets printed second";

BEGIN {
        print "This gets printed first";
}

print "This gets printed third";
```

A BEGIN block is always executed first. If you create multiple BEGIN blocks (don't), they are executed in order from top to bottom as the compiler encounters them. A BEGIN block always executes first even if it is placed halfway through a script (don't do this) or at the end (or this). Do not mess with the natural order of code. Put BEGIN blocks at the beginning!

A BEGIN block is executed as soon as the block has been parsed. Once this is done, parsing resumes at the end of the BEGIN block. Only once the whole script or module has been parsed is any of the code outside of BEGIN blocks executed.

BEGIN总是最先执行的. 如果你创建了多个BEGIN块(别~), 它们是按顺序从上往下执行的. 不管BEGIN是在中间还是末尾(别~)都是最先执行的. 不要跟普通的代码次序混淆了. 把BEGIN放在最前面吧.

BEGIN块是一解析到就会执行的. BEGIN执行完之后, 解析会接着执行. 只有当BEGIN执行完, 这个文本或模块解析完, BEGIN之外的代码才会执行.

```perl
use strict;
use warnings;

print "This 'print' statement gets parsed successfully but never executed";

BEGIN {
        print "This gets printed first";
}

print "This, also, is parsed successfully but never executed";
```

...because e4h8v3oitv8h4o8gch3o84c3 there is a huge parsing error down here.

Because they are executed at compilation time, a BEGIN block placed inside a conditional block will still be executed first, even if the conditional evaluates to false and despite the fact that the conditional has not been evaluated at all yet and in fact may never be evaluated.

因为这里有个很大的解析错误.

因为BEGIN快是在编译的时候执行的, if表达式内的BEGIN快还是会首先执行, 不管if的条件是不是假的也不管这个条件求值了没(实际上可能是永远不会求值的).

```perl
if(0) {
        BEGIN {
                print "This will definitely get printed";
        }
        print "Even though this won't";
}
```

Do not put BEGIN blocks in conditionals! If you want to do something conditionally at compile time, you need to put the conditional inside the BEGIN block: 

不用把BEGIN放在条件语句里! 如果你想在编译的时候做一些条件分支, 把条件分支放到BEGIN块里:

```perl
BEGIN {
        if($condition) {
                # etc.
        }
}
```

# use

Okay. Now that you understand the obtuse behaviour and semantics of packages, modules, class methods and BEGIN blocks, I can explain the exceedingly commonly-seen use function.

The following three statements:

好了, 你们已经了解了包, 模块, 类方法还有BEGIN的语义和很挫的行为, 我可以解释非常常见的`use`函数了.

看下面三个表达式:

```perl
use Caterpillar ("crawl", "pupate");
use Caterpillar ();
use Caterpillar;
```

are respectively equivalent to:

分别相当于:

```perl
BEGIN {
        require Caterpillar;
        Caterpillar->import("crawl", "pupate");
}
BEGIN {
        require Caterpillar;
}
BEGIN {
        require Caterpillar;
        Caterpillar->import();
}
```

- No, the three examples are not in the wrong order. It is just that Perl is dumb.
不是啊, 这三个例子的顺序没错(对?)啊. 不过Perl比较傻.
- A use call is a disguised BEGIN block. The same warnings apply. use statements must always be placed at the top of the file, and never inside conditionals.
`use`调用其实是伪装的BEGIN块. 警告是一样的. `use`语句也要放在文件头, 不要放在条件分支里.
- import() is not a built-in Perl function. It is a user-defined class method. The burden is on the programmer of the Caterpillar package to define or inherit import(), and the method could theoretically accept anything as arguments and do anything with those arguments. use Caterpillar; could do anything. Consult the documentation of Caterpillar.pm to find out exactly what will happen.
'import()'不是内置的Perl函数, 是自定义的类方法. 定义或者继承`important()`方法就是写Caterpillar包的程序员的事情了, 而且理论上这个方法是可以接收任意的参数并用这些参数做任意的事情. `use Caterpillar;`可以做任意的事情. 欲知详情, 请查阅`Caterpillar.pm`的文档.
- Notice how require Caterpillar loads a module named Caterpillar.pm, whereas `Caterpillar->import()` calls the `import()` subroutine that was defined inside the Caterpillar package. Let's hope the module and the package coincide!
要注意`require Caterpillar`是怎么载入`Caterpillar.pm`这个模块的, 而`Caterpillar->important()`调用的是`Caterpillar`包里定义的子函数`import()`. 但愿这个模块和包是一致的.

# Exporter

The most common way to define an `import()` method is to inherit it from the Exporter module. Exporter is a core module, and a de facto core feature of the Perl programming language. In Exporter's implementation of `import()`, the list of arguments that you pass in is interpreted as a list of subroutine names. When a subroutine is `import()`ed, it becomes available in the current package as well as in its own original package.

This concept is easiest to grasp using an example. Here's what Caterpillar.pm looks like:

定义`import()`最常用的方式是从`Exporter`模块继承. Exporter是核模块, 也是Perl语言的实际上的核心特性. 在Exporter的`import()`实现里, 你输入的参数集会被解释成一组子函数的名字. 当一个子函数通过`import()`调用了, 在当前的包里也就可用了.

这个概念用个例子就很好掌握了. 下面就是`Caterpillar.pm`:

```perl
use strict;
use warnings;

package Caterpillar;

# Inherit from Exporter
use parent ("Exporter");

sub crawl  { print "inch inch";   }
sub eat    { print "chomp chomp"; }
sub pupate { print "bloop bloop"; }

our @EXPORT_OK = ("crawl", "eat");

return 1;
```

The package variable `@EXPORT_OK` should contain a list of subroutine names.

Another piece of code may then `import()` these subroutines by name, typically using a use statement:

包里的变量`@EXPORT_OK`要包含一些子函数名.

然后其他的代码就可以通过函数名用`import()`导入这些子函数, 典型的是用`use`语句:

```perl
use strict;
use warnings;
use Caterpillar ("crawl");

crawl(); # "inch inch"
```

In this case, the current package is main, so the `crawl()` call is actually a call to `main::crawl()`, which (because it was imported) maps to `Caterpillar::crawl()`.

Note: regardless of the content of `@EXPORT_OK`, every method can always be called "longhand":

在这里, 当前的包为`main`, 所以`crawl()`就是`main::crawl()`, 这里映射到了`Caterpillar::crawl()`.

注: 不管`@EXPORT_OK`的内容的话, 每个方法都可以常规的调用:

```perl
use strict;
use warnings;
use Caterpillar (); # no subroutines named, no import() call made

# and yet...
Caterpillar::crawl();  # "inch inch"
Caterpillar::eat();    # "chomp chomp"
Caterpillar::pupate(); # "bloop bloop"
```

Perl has no private methods. Customarily, a method intended for private use is named with a leading underscore or two.

Perl是没有私有方法的. 但习惯上, 要打算把一个方法当私有方法用的话就是在命名的时候以一个或俩个下划线开头.

## @EXPORT

The Exporter module also defines a package variable called `@EXPORT`, which can also be populated with a list of subroutine names.

Exporter模块定义了一个`@EXPORT`变量, 这个变量也可以用一组子函数名.

```perl
use strict;
use warnings;

package Caterpillar;

# Inherit from Exporter
use parent ("Exporter");

sub crawl  { print "inch inch";   }
sub eat    { print "chomp chomp"; }
sub pupate { print "bloop bloop"; }

our @EXPORT = ("crawl", "eat", "pupate");

return 1;
```

The subroutines named in `@EXPORT` are exported if `import()` is called with no arguments at all, which is what happens here:

如果`import()`调用的时候没有参数的话, `@EXPORT`里面的子函数就会被导出, 就像这样:

```perl
use strict;
use warnings;
use Caterpillar; # calls import() with no arguments

crawl();  # "inch inch"
eat();    # "chomp chomp"
pupate(); # "bloop bloop"
```

But notice how we are back in a situation where, without other clues, it might not be easy to tell where `crawl()` was originally defined. The moral of this story is twofold:

但要注意我们回到了之前的那种——没有其他线索, 说不出`crawl()`是在哪里定义的哪种境地. 这个故事的寓意有两层:

1. When creating a module which makes use of Exporter, never use `@EXPORT` to export subroutines by default. Always make the user call subroutines "longhand" or `import()` them explicitly (using e.g. use Caterpillar ("crawl"), which is a strong clue to look in Caterpillar.pm for the definition of `crawl()`).
当用Exporter新建一个模块的时候, 千万不要默认的用`@EXPORT`来导出子函数, 要显式的用常规调用或者用`import()`导入(用法示例——`use Caterpillar("crawl")`, 这就强烈的指明了要去`Caterpillar.pm`里面找`crawl()`的定义).
2. When useing a module which makes use of Exporter, always explicitly name the subroutines you want to `import()`. If you don't want to `import()` any subroutines and wish to refer to them longhand, you must supply an explicit empty list: `use Caterpillar ()`.
当`use`的模块里用到Exporter的时候, 要显式的命名你要`import()`导出的子函数. 如果你不想导出子函数, 还希望可以正常的引用它们, 就要用显示的空列表: `use Caterpillar ()`.

# Miscellaneous notes 杂项

- The core module `Data::Dumper` can be used to output an arbitrary scalar to the screen. This is an essential debug tool.
核模块`Date::Dumper`可以用来输出一个霸气的标量到屏幕. 这是个重要的调试工具.
- There's an alternate syntax, `qw{ }`, for declaring arrays. This is often seen in use statements: `-ccount qw{create open close suspend delete};`
还有另外一种语法来定义数组, `qw{}`. 这个在`use`语句里比较常用: `-ccount qw{create open close suspend delete};`
- In `=~ m//` and `=~ s///` operations, you can use braces instead of slashes as the regex delimiters. This is quite useful if your regex contains a lot of slashes, which would otherwise need escaping with backslashes. For example, `=~ m{///}` matches three literal forward slashes, and `=~ s{^https?://}{}` removes the protocol part of a URL.
在`=~ m//`和`=~ s///`操作里, 你可以用大括号来代替斜杠作正则表达式分隔符. 这个在你的正则表达式里包含很多斜杠的时候很有用, 不然的话你就要很多反斜杠了. 举例来说, `=~ m{///}`匹配三个斜杠字符, `=~ s{^https?://}{}`来去掉URL里的协议部分.
- Perl does have CONSTANTS. These are discouraged now, but weren't always. Constants are actually just subroutine calls with omitted brackets.
Perl是没有常数的. 这个现在是不鼓励的, 但不会一直这样. 常量实际上就是没有括号的调用子函数.
- Sometimes people omit quotes around hash keys, writing `$hash{key}` instead of `$hash{"key"}`. They can get away with it because in this situation the bareword key occurs as the string "key", as opposed to a subroutine call key().
有时候人们会省略哈希值里的引号, 把`$hash{"key"}`写成`$hash{key}`. 其实可以不这么用, 因为这里词`key`是以字符串想形式出现的, 这根调用`key()`是相反的.
- If you see a block of unformatted code wrapped in a delimiter with double chevrons, like <<EOF, the magic word to Google for is "here-doc".
如果你看到像是`<<EOF`一样的没有格式化的包在俩箭头号分隔符里的代码段, 用关键字"here-doc"去Google把.
- Warning! Many built-in functions can be called with no arguments, causing them to operate on `$_` instead. Hopefully this will help you understand formations like:
注意! 很多内建函数可以不加参数就调用, 这样会导致它们作用于`@_`. 希望这个能帮你了解像这样的结构:

```perl
print foreach @array;
```

and

还有,

```perl
foreach ( @array ) {
        next unless defined;
}
```

I dislike this formation because it can lead to problems when refactoring.

我不喜欢这样的结构, 因为这个会在重构时造成麻烦.

And that's two and a half hours.

这就是两个半小时.

-----

翻译完了, 用了整整两天. 感觉翻译的又臭又长, 看着网页进度条上细细的进度块, 在想是不是应该把这篇分个几份出来.

还有就是**2 hours and a half**, 完全是被坑的感觉. 说实话这篇文章应该还是把Perl讲全了的, 扫一眼的话当然还是很快能看完. 不过要想认真学的话至少都要跟着敲敲示例代码体会体会细节, 2个半小时, 正则表达式都学不完啊.

