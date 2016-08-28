title: 《Learning PERL》笔记
date: 2015-4-28 11:16:23
---

[TOC]

# scalar和array的记法

```perl
$calar --> scalar
@rray --> array
```

还是挺巧妙的, 不过`%hash`就没什么的了.

# 字符串tricks

## 犀利的qw

qw表示quoted words：

```perl
qw(fred barney betty wilma dino)
# equals ("fred", "barney", "betty", "wilma", "dino")
```

此外, 用了qw就不限于用括号序对了, 可以用的序对符有:

```perl
qw!...!
qw/.../
qw#...#
qw{...}
qw[...]
qw<...>
...
```

好灵活的。


## a..b递增数字序列

生成`(a, a+1, a+2,..., b-1, b)`,这里当然b是要大于a的. 测试了一下, a大于b的话返回的是一个空列表.

<!--more-->


## 字符串插值

`"@array"`与`'@array'`是不同的, 前者会进行列表值的插入展开, 而后者则是原始的literal字符串不进行展开.

插值的形式不止于此, 比如`"\Lstr"`会将`str`全都变成小写, 相应的`"\Ustr"`会将字符串全都变成大写字母.

## 字符串duplicate

```perl
"fred"x3 # 'x' not '*'
>"fredfredfred"
```

相当于python里的`str*n`, PERL在这方面果然不会落后.

## 格式化字符串

用的是`%`:

- `%g`, 指定自动数字形式;
- `%-n`, 指定宽度为n个字符, 左对齐;
- `%n`, 指定宽度为n个字符, 右对齐;
- `%%`, 输出百分号%

可变宽度的字符串可以用:

```perl
printf "%${width}s..." $str
printf "%*s..." ,$width, $str
```

`sprintf`函数也接收格式化字符串, 不过不打印出来, 而是用于保存到变量里.

这样的两种形式.

# `$#<list_name>`列表长度(-1)

返回的是list的个数-1,因此可以使用`$list_name[$#list_name]`来获取列表的最后一个值.

# build-in函数杂计

## splice

原型是: 

```perl
splice (arg1, arg2[, arg3[, arg4]])
#instance
@remove = splice(arg1, ...)
```

arg1为目标列表, arg2为起始位置(从1开始计), arg3为作用的长度, arg4为替换进去的列表.
示例中`@remove`为切出的列表, `arg1`为作用后的结果列表, 与直觉不相符呢.

而且要注意的是在列表遍历中(典型的如foreach)中对遍历成员的操作会影响目标列表, 这个或许是splice的这种行为的原因.

## `unlink`和`glob`

`unlink`和`glob`相当于shell中的`rm`和`glob`, 即`unlink glob '*.o'`就相当于在shell中执行`rm *.o`, 不过推荐的执行方式如下:

```perl
foreach my $file (glob "*.o"){
    unlink $file or warn "...$!";
}
```

## `index`和`rindex`

直接看例子:

```perl
$location_int = index @heap, $needle;
```

`$location_int`为`@heap`中首次出现`$needle`的位置.

还有个`rindex`是反向查找.

## `grep`

`grep`跟shell里的`grep`没有关系, 其实跟FP里的`filter`是一个意思, 直接看代码示例吧:

```perl
my @odd_number = grep {$_%2} 1..1000;
my @matching_lines = grep /\bfred\b/i, <$fh>;
my $matching_lines_count = grep /\bfred\b/i, <$fh>;
```

最后一条加了一个context变换的trick而已.

# 特殊变量

## `$_`默认变量

直接上代码好了:

```perl
foreach (1..10) {
    print $_;
}
#12345678910
$_ = "hahah"
print;
#hahah
```

`$_`在这里起到了一个表达式"管道"的作用.

## 函数中的`@_`

`@_`是函数的私有变量, 表示函数的参数列表, 可以用`$_[n]`来进行逐个索引访问.

`@_`跟默认变量`$_`是没有任何关系的.

## 命令行参数`@ARGV`与钻石操作符`<>`

命令行的参数保存在`@ARGV`里. 

`<>`将`@ARGV`中指定全部文件打开并将文件句柄全都存储在`<>`内, 当然`@ARGV`也可以显式的认为修改. 如果`@ARGV`中没有参数, 则`<>`使用`<STDIN>`.

```perl
while (<>) {
    print;
}
```

## `$!`

`$!`中存放着系统错误信息.

上面的代码等价于一个`cat`命令, 当然这个命令可以写的更简单一点: `print <>`.

## `$1, $2, ... $n`

为正则表达式匹配之后的分组内容捕获, 各个变量表示分组匹配的内容.

## 正则表达式默认匹配变量

`$(apostrophe)$&$'`为正则表达式匹配之后的默认变量, $\`表示匹配内容之前的字符串, `$&`表示匹配的字符串, `$'`表示匹配内容之后的字符串.

## `$^I`备份文件变量

用`$^I = "*.bak"`这样的形式来设置备份文件.

## `$a`和`$b`

排序子程序中的默认变量, 表示要进行比较的两个输入.

# 上下文context

`@array`在列表上下文中返回列表的列表项目值, 在标量上下文中返回列表元素的个数, `scalar @array`会强制制造标量上下文, 返回`@array`的列表元素个数.

# `<STDIN>`标准输入

`<STDIN>`其实是一个句柄啦, 指向标准输入, 也就是键盘, 标准用法是这样的:
    
    chomp(@lines = <STDIN>);

表示读取C-D之前的所有键盘输入内容, 每行去掉换行符然后存到列表`@lines`里面.

# 函数

## 定义与调用

定义格式为:

```perl
sub sub_name{
...
}
```

调用的格式为:

```perl
&sub_name;
```

在调用时, 如果没有跟build-in函数名冲突, 可以不用加`&`符号(build-in函数是不用加符号的). 其实这种设置灵活是零活, 但是还是都加`&`的好.

另外, 函数结束前执行的最后一步的结果会作为返回值, 除非显式的使用`return`. 这个的话依然还是统一用`return`比较好.


## `my`创建变量

使用`my`可以创建作用域内的变量, 典型的就是函数内部啦:

```perl
sub sub_name{
    my ($m, $n) = @_;
    ...
}
```

此外, 还可以用于if, while, foreach的作用域内, 比如:

```perl
foreach my $rock (@rocks){
...
}
```

## `state`

用`state`创建的是函数的**静态变量**.

# for和foreach

`foreach`是`for`的一个子集, `for(@args)`和`foreach(@args)`是可以互相替换的, 此外`for`的另外的用法就是常见的`for($i = 0; $i < n; $i++)`.

# 文件句柄

```perl
open CONFIG, 'dino'; #default mode(read)
open OCNFIG, '<dino'; #read only mode
open REDROCK, '>fred'; #write mode, create file if not exists
open LOG, '>>logfile'; #append mode, create file if not exists

print LOG "log info..."
select LOG
print "log info"
```

要注意使用的文件符为单引号, 读写指定符跟命令行的管道重定向符号是一致的. 往文件里面添加内容直接使用第6行的格式即可.

`select`可以选择默认句柄, 用来取代`STDIN`和`STDOUT`.

# hash

`%var`表示一个哈希变量.

## hash的reverse

    my %inverse_hash = reverse %any_hash;
    
使原来的k-v对转换为v-k对, 根据hash的默认属性, 这样翻转需要原先的v中没有重复值, 不然就会出现转换后k值的覆盖.

## keys, values函数

分别返回哈希表的键列表和值列表:

```perl
foreach $key (sort keys %hash) {
    $hash{$key} = ...
    ...
}
```

另外`each`可以直接返回键值对:

```perl
while(($k, $v) = each %hash)
```

# 正则表达式

## 属性元字符

形式为: `\p{property}`, 比较重要的几个列举如下:

- `\p{Space}`: 表示Unicode中定义的空格;
- `\p{Digit}`: 表示Unicode中定义的数字;
- `\p{Hex}`: 表示16进制字符, 相当于`[0-9A-Fa-f]`;
- `\P{property}`: 表示反义;

具体的内容查阅`perluniprops`吧.

## 括号分组

正则表达式中的组号以左括号从左至右的顺序从1开始编号, `\1, \2, ... \n`为正则表达式中的引用, `$1, $2, ...$n`为正则表达式匹配之后的组内容捕获.

另外可以使用`\g{n}`这样的形式, 以消除`\111`这样的歧义(`\111`表示第111组, 还是第1组后面跟11还是表示11组后面跟1呢?).

更研究的方法是用label来命名分组--`(?<label_name>pattern)`, 然后用`\g{label_name}`引用, 用`${label_name}`来获取捕获内容.

`(?:pattern)`表示不占用组号的分组.

## 默认匹配变量

`$(apostrophe)$&$'`为正则表达式匹配之后的默认变量, $\`表示匹配内容之前的字符串, `$&`表示匹配的字符串, `$'`表示匹配内容之后的字符串, 一个可以用来做正则匹配调试的程序如下:

```perl
while(<>){
    chomp;
    if (/pattern/){
        print "matched: |$`<$&>$'\n";
    } else {
        print "No match: |$_|\n";
    }
}
```

## 匹配选项

写在正则表达式的最后面--`/pattern/option_mark`, 常用的选项有:

- `/g`: 获取所有的匹配, 即返回到一个array而不是scalar里;
- `/i`: 忽略大小写;
- `/s`: 使`.`匹配`\n`;
- `/x`: 忽略正则表达式中的空格, 使用这个选项可以写出可能性明显提升的正则表达式, 因为使用了这个选项就可以对正则表达式进行断行了, 并且可以在每行都写上注释.

# 单行流例程

```perl
[usr@comp ~]$ perl -p -i.bak -w -e 's/Randall/Randal/g' *.dat

#!/usr/bin/perl -w
$^I = ".bak";
while (<>) {
    s/Randall/Randal/g;
    print;
}
```

`-p`相当于是:

```perl
while (<>) {
    "-e stuff"
    print;
}
```

意思是打印每一行, `-e`指定的内容就填充到`print`这一句之上.

`-w`相当于是代码中的第一行, 打开warning开关的意思, `-i.bak`就相当于是`$^I = ".bak"`设置同名备份文件的意思了.

# 模块使用

```perl
use File::Basename qw/.../
```

表示引入模块中的`...`函数, 为空的话表示不引入.

## Try::Tiny

用了这个模块就可以用`try catch`了. 不过这个模块需要从CPAN中下载.

## List::Util

标准库中的列表处理库, 里面有`shuffle(@list)`这种函数, 应该很实用.

# 排序子程序

```perl
sub by_name {
    if ($a < $b) -1;
    elsif ($a > $b) 1;
    else 0;
}
```

上面的这个就是一个排序程序, `$a`和`$b`表示顺序赋值的两个输入. 上面的程序用于`sort`的排序子程序中, 相当于`sort`的一个callback.

上面的程序表示按数值从大往小排序, 应用方式为:

    my @result = sort by_name @some_num;

可以替换为:

    my @result = sort {$a <=> $b} @some_num;

上面的那个排序程序也可以简化为:

    sub by_name {$a <=> $b};

按字符排序的子程序写法是:

    sub by_code_point {$a cmp $b}
    
这个就是`sort`的默认行为.

进行反向排序的话就交换一下两个排序变量的顺序就可以了.

ps: 排序子函数的函数名用`by_`前缀的形式似乎是个convention.

## 哈希表多重排序应用

```perl
my %score; # a {name:score} hash

sub by_score_and_name {
    $score{$b} <=> $score{$a}
    or
    $a cmp $b
}
```

上面的排序程序表示先按分数从小到大排序, 同分的话再按字母排序.

# 数组切片和哈希切片

直接上代码说明, 数组切片:

```perl
my @names;
...
my ($first, $last) = (sort @names)[0, -1];
```

哈希切片:

```perl
my %score;
...
my @three_scores = @score{qw/barney fred dino/};
```

要注意哈希的上下文.







