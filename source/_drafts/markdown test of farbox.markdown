title: 慢慢琢磨Farbox
Status: Public
date: 2013-8-28 11:16:33
---


[TOC]

很明显的一点是farbox的markdown是定制过的，比如支持表格了，还有脚注。还有就是代码关键字高亮，是用的另外的引擎。

而且，这个跟别处的还是有些不同，就像这里第一段是用全角空了俩格的，但是显示不出来。

<!--more-->

farbox的写作规则里面有讲到文章属性——标题、修改时间、访问属性这些的设置规则，就是在文件的顶部用`key: value`这样的形式设置，跟org-mode越来越接近了。这个还是蛮方便的，可以设置的属性有标题(Title)、日期(Date)、访问属性(Status)——public\Draft\secret\private\others还有文章链接(url)和标签(Tags)。

不过话说回来，这些特性的不统一必然会造成“移植”方面的问题。看来markdown至少要引入注释系统才行。

-----
2013-9-1 15:12:53: 目前，Farbox已经可以支持公式了，这样代码、表格、公式，工科文档的全部所需都已经完备了，很不错。

----- 
`\(x_1 = 132\)`
分割线来一个

TOC——table of content

这个toc要放在顶格，这个要注意一下。。。

\[PAGE]

# header1

>床前明月光
疑是地上霜
举头望明月
低头思故乡

## header2

_床前明月光_
__床前明月光__
___床前明月光___
疑是地上霜
举头望明月
低头思故乡[^1] 





![Image Title](/_image/2013-08-14/1.jpg?width=320&height=320)

### header3
[a link it is](www.baidu.com)
#### 黑喂狗。。。

```c
#define ON 1
#include "stdio.h"

struct sig
{
	double amp;
	double pha;
}

int main()
{
	printf("hi there");
	return 0;
}

```
```verilog
`include "define.v"

module(
	address_bus,data_bus,cs,
	wr,rd
);
	input [7:0] address_bus;
	inout [7:0] data_bus;
	input cs,wr,rd;

	assign
		data_bus=((cs==`LOW)&&(rd==`HIGH))?data_out:8'hz;

	reg [7:0] data_out;
	always @(*)
	begin
		if((cs==`LOW)&&(rd==`HIGH))
		begin
			data_out=address_bus+8'd1;
		end
	end		
endmoule
```
```scheme
(add-hook 'verilog-mode-hook
	(lambda () (auto-complete-mode)))

(setq hightlight-tail-colors
	'(("black" . o)
		("#bc2525" . 25)
		("black" . 66)))
```
	:::scheme

(add-hook 'verilog-mode-hook
	(lambda () (auto-complete-mode)))

(setq hightlight-tail-colors
	'(("black" . o)
		("#bc2525" . 25)
		("black" . 66)))

##### 哟哟哟

- list is sick
	- really
	- really
	- really sick
- you can not break
lines
----- 

Monday|Tuesday|Wednesday|Thursday|Friday|Saturday
---|---|---|---|---|---
work|work|work|work|work|work
work|work|work|work|work|work
work|work|work|work|work|work

-----

# mathjax测试下 $rock \Tex{}$

## 希腊字母

\alpha \beta \gamma \delta \epsilon \varepsilon \zeta \eta \Gamma \Delta \Theta

\theta \vartheta \iota \kappa \lambda \mu \nu \xi \Lambda \Xi \Pi

o \pi \varpi \rho \varrho \sigma \varsigma \Sigma \Upsilon \Phi 

\tau \upsilon \phi \varphi \chi \psi \omega \Psi \Omega

```mathjax
\alpha \beta \gamma \delta \epsilon \varepsilon \zeta \eta \Gamma \Delta \Theta
\theta \vartheta \iota \kappa \lambda \mu \nu \xi \Lambda \Xi \Pi
o \pi \varpi \rho \varrho \sigma \varsigma \Sigma \Upsilon \Phi 
\tau \upsilon \phi \varphi \chi \psi \omega \Psi \Omega
```

## 其他


\[x_{ij}^2\quad \sqrt[2]{x}\]

```mathjax
\[x_{ij}^2\quad \sqrt[2]{x}\]
```

\[\pm \times \div \cdot \cap \cup \geq \leq \neq \approx \equiv\]
```mathjax
\[\pm \times \div \cdot \cap \cup \geq \leq \neq \approx \equiv\]
```

\sum_{i=1}^n i \prod_{i=1}^n \lim_{x\to0}x^2 \int_a^b x^2 dx
\[\sum_{i=1}^n i \prod_{i=1}^n \lim_{x\to0}x^2 \int_a^b x^2 dx\]
```mathjax
$\sum_{i=1}^n i \prod_{i=1}^n \lim_{x\to0}x^2 \int_a^b x^2 dx$
\[\sum_{i=1}^n i \prod_{i=1}^n \lim_{x\to0}x^2 \int_a^b x^2 dx\]
```

-----

# MathJax新发现

我现在发现[Farbox网站](www.farbox.com)上面其实是不支持文档上面说的```mathjax插入公式语法的，而是支持\$TeX{}\$插入行内公式和\$\$TeX{}\$\$独立行公式这种格式，但是本地的是支持代码定义式的插入的，不过这个我也不知道是个什么原理。在<head>间插入的脚本还是一样的:

```html
<script src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML-full.js" type= "text/javascript">
   MathJax.Hub.Config({
	   tex2jax: {
			inlineMath: [ ['$','$'], ['\\(','\\)'] ]
			},
		extensions: ["jsMath2jax.js", 'tex2jax.js'],
		messageStyle: "none"
	});
</script>
```

所以还是再来一遍吧。

## 希腊字母

\alpha \beta \gamma \delta \epsilon \varepsilon \zeta \eta \Gamma \Delta \Theta

\theta \vartheta \iota \kappa \lambda \mu \nu \xi \Lambda \Xi \Pi

o \pi \varpi \rho \varrho \sigma \varsigma \Sigma \Upsilon \Phi 

\tau \upsilon \phi \varphi \chi \psi \omega \Psi \Omega

$$\alpha \beta \gamma \delta \epsilon \varepsilon \zeta \eta \Gamma \Delta \Theta$$
$$\theta \vartheta \iota \kappa \lambda \mu \nu \xi \Lambda \Xi \Pi$$
$$o \pi \varpi \rho \varrho \sigma \varsigma \Sigma \Upsilon \Phi$$
$$\tau \upsilon \phi \varphi \chi \psi \omega \Psi \Omega$$

## 其他


\[x_{ij}^2\quad \sqrt[2]{x}\]

$$x_{ij}^2\quad \sqrt[2]{x}$$

\[\pm \times \div \cdot \cap \cup \geq \leq \neq \approx \equiv\]

$$\pm \times \div \cdot \cap \cup \geq \leq \neq \approx \equiv$$

\sum_{i=1}^n i \prod_{i=1}^n \lim_{x\to0}x^2 \int_a^b x^2 dx
\[\sum_{i=1}^n i \prod_{i=1}^n \lim_{x\to0}x^2 \int_a^b x^2 dx\]

$$\sum_{i=1}^n i \prod_{i=1}^n \lim_{x\to0}x^2 \int_a^b x^2 dx$$
$$\sum_{i=1}^n i \prod_{i=1}^n \lim_{x\to0}x^2 \int_a^b x^2 dx$$

# MathJax的新新发现。。。

原来MathJax还是支持公式自动编号的，用的还是$\Tex$自带的语法，感觉越来越博大精深了。

先贴一个可以用自动公式编号的脚本：

```html
	<script type="text/x-mathjax-config">
	    MathJax.Hub.Config({
		    tex2jax: {
			    inlineMath: [
				    ['$', '$'],
					['\\(', '\\)']
				],
				displayMath: [
					['$$', '$$'],
					["\\[", "\\]"]
				],
				processEscapes: true
			},
			TeX: {
				extensions: ["AMSmath.js", "AMSsymbols.js"],
				equationNumbers: {
					autoNumber: ["AMS"],
					useLabelIds: true
				},
				Macros: {
					hfill: "{}"
				}
			},
			"HTML-CSS": {
				linebreaks: {
					automatic: true
				},
				availableFonts: ["TeX"],
				scale: 110
			},
			SVG: {
				linebreaks: {
					automatic: true
				}
			}
		});
	</script>
	<script type="text/javascript" src="https://c328740.ssl.cf1.rackcdn.com/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
    
	</script>
```

然后就可以用:
```
	$$
	\begin{equation}
		${1:your equation}
	\end{equation}
	$$
```
插入公式了，这个可以插很多个公式，每个公式之后可以用`\newline`换行（`\\`不能换行，不知道为啥），然后就出来带编号的公式了。

我试了一下，公式对齐也是可以用的,就是用的Tex的`align`语法，用`&`标记对齐位置：

```
$$
\begin{align}
	W_N^{(k+N)n}&=W_N^{k(n+N)}=W^{kn}\newline
	W_N^{k(N-n)}&=W_N^{-kn}=(W_N^kn)^*\newline
	W_N^{k+\frac{N}{2}}&=-W_N^k
\end{align}
$$
```
$$
\begin{align}
	W_N^{(k+N)n}&=W_N^{k(n+N)}=W^{kn}\newline
	W_N^{k(N-n)}&=W_N^{-kn}=(W_N^kn)^*\newline
	W_N^{k+\frac{N}{2}}&=-W_N^k
\end{align}
$$


[^1]: 静夜思·李白~

