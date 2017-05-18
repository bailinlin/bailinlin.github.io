---
title: markdown-cheatsheet
date: 2017-04-25 16:15:57
tags:
---
> 文档的书写是大家的必修课,从大学时的论文,到工作之后的项目文档,个人博客,文档是每个人生活中都需要接触到的
文档的优雅程度直接关系到读者的阅读幸福指数.不过现在有好多程序员居然不会 markdown 语法,其实真的超级简单,
下面是我翻译的一篇 markdown 入门教程 [原文地址](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet)

下面的内容是 markdown 一个快速引导和使用举例.如果你想获得更多的信息,你可以去看 [ John Gruber 的原始文档](http://daringfireball.net/projects/markdown/)
或则是看 `github 上的 markdown 语法信息`(译者:这个现在已经不被维护了,文章链接已经找不到了)

这篇文章主要讲的是你正在寻找的 markdown 的一些特殊的小技能,如果你想了解更多,可以点击 [更多 markdown 小技巧](https://github.com/adam-p/markdown-here/wiki/Other-Markdown-Tools)

### 内容罗列(目录)

1. [Header](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#headers)
2. [Emphasis](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#emphasis)
3. [Lists](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#lists)
4. [Links](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#links)
5. [Images](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#images)
6. [Code and Syntax Highlighting](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#code)
7. [Blockquotes](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#blockquotes)
8. [Inline HTML](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#html)
9. [Horizontal Rule](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#hr)
10. [Line Breaks](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#lines)
11. [Youtube videos](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#videos)

## Header,标题

<pre>
    # H1
    ## H2
    ### H3
    #### H4
    ##### H5
    ###### H6

    不太一样的是,H1 和 H2 有下划线的样式

    Alt-H1
    ======

    Alt-H2
    ------
</pre>

# H1
## H2
### H3
#### H4
##### H5
###### H6

    不太一样的是,H1 和 H2 有下划线的样式

Alt-H1
======

Alt-H2
------

## Emphasis,重要性标记

> 译者:重要标记的语法就是把需要提现重要的文字加粗,即放在两个<code> * </code>中间


<pre>
    Emphasis, aka italics, with *asterisks* or _underscores_.

    Strong emphasis, aka bold, with **asterisks** or __underscores__.

    Combined emphasis with **asterisks and _underscores_**.

    Strikethrough uses two tildes. ~~Scratch this.~~
</pre>

Emphasis, aka italics, with *asterisks* or _underscores_.

Strong emphasis, aka bold, with **asterisks** or __underscores__.

Combined emphasis with **asterisks and _underscores_**.

Strikethrough uses two tildes. ~~Scratch this.~~

## Lists 列表

 在下面的例子中哦,我们能学到用一串的空格和点来展示列表

 <pre>
    1. First ordered list item
    2. Another item
    ⋅⋅* Unordered sub-list.
    1. Actual numbers don't matter, just that it's a number
    ⋅⋅1. Ordered sub-list
    4. And another item.

    ⋅⋅⋅You can have properly indented paragraphs within list items. Notice the blank line above, and the leading spaces (at least one, but we'll use three here to also align the raw Markdown).

    ⋅⋅⋅To have a line break without a paragraph, you will need to use two trailing spaces.⋅⋅
    ⋅⋅⋅Note that this line is separate, but within the same paragraph.⋅⋅
    ⋅⋅⋅(This is contrary to the typical GFM line break behaviour, where trailing spaces are not required.)

    * Unordered list can use asterisks
    - Or minuses
    + Or pluses
 </pre>

 1. First ordered list item
 2. Another item
 ⋅⋅* Unordered sub-list.
 1. Actual numbers don't matter, just that it's a number
 ⋅⋅1. Ordered sub-list
 4. And another item.

 ⋅⋅⋅ You can have properly indented paragraphs within list items. Notice the blank line above, and the leading spaces (at least one, but we'll use three here to also align the raw Markdown).

 ⋅⋅⋅ To have a line break without a paragraph, you will need to use two trailing spaces.⋅⋅
 ⋅⋅⋅ Note that this line is separate, but within the same paragraph.⋅⋅
 ⋅⋅⋅ (This is contrary to the typical GFM line break behaviour, where trailing spaces are not required.)

 * Unordered list can use asterisks
 - Or minuses
 + Or pluses


## Links 链接

 我们有两种方法来创建链接

<pre>

    [I'm an inline-style link](https://www.google.com)

    [I'm an inline-style link with title](https://www.google.com "Google's Homepage")

    [I'm a reference-style link][Arbitrary case-insensitive reference text]

    [I'm a relative reference to a repository file](../blob/master/LICENSE)

    [You can use numbers for reference-style link definitions][1]

    Or leave it empty and use the [link text itself].

    URLs and URLs in angle brackets will automatically get turned into links.
    http://www.example.com or <http://www.example.com> and sometimes
    example.com (but not on Github, for example).

    Some text to show that the reference links can follow later.

    [arbitrary case-insensitive reference text]: https://www.mozilla.org
    [1]: http://slashdot.org
    [link text itself]: http://www.reddit.com

</pre>

[I'm an inline-style link](https://www.google.com)

[I'm an inline-style link with title](https://www.google.com "Google's Homepage")

[I'm a reference-style link][Arbitrary case-insensitive reference text]

[I'm a relative reference to a repository file](../blob/master/LICENSE)

[You can use numbers for reference-style link definitions][1]

Or leave it empty and use the [link text itself].

URLs and URLs in angle brackets will automatically get turned into links.
http://www.example.com or <http://www.example.com> and sometimes
example.com (but not on Github, for example).

Some text to show that the reference links can follow later.

[arbitrary case-insensitive reference text]: https://www.mozilla.org
[1]: http://slashdot.org
[link text itself]: http://www.reddit.com

URLs and URLs in angle brackets will automatically get turned into links. http://www.example.com or http://www.example.com and sometimes example.com (but not on Github, for example).

Some text to show that the reference links can follow later.

urls 和被包括在宗括号的 urls 会被自动的解析成链接,`http://www.example.com `或则` http://www.example.com` 或则 `example.com`(并非 github 上真正的链接,就是用来举例子)

##Images 图片

<pre>

    这个是我们的图片(鼠标浮动到上面来看图片的标题文案)

    行内写法:
    ![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

    引用写法:
    ![alt text][logo]

    [logo]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 2"

</pre>

这个是我们的图片(鼠标浮动到上面来看图片的标题文案)

    行内写法:
    ![alt text](https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 1")

    引用写法:
    ![alt text][logo]

    [logo]: https://github.com/adam-p/markdown-here/raw/master/src/common/images/icon48.png "Logo Title Text 2"

## Code and Syntax Highlighting 代码和语法高亮


展示代码块是 markdown 语法中的一部分,但是显示语法高亮不是他的工作.我们可以用`renderers`(渲染语法高粱的方式或则说是编译器),就像 github 内置的 markdown 编辑器,就支持语法高亮.但是不同的渲染器对于那些语法的高亮和如何命名高亮的语言类型都是有差异的,github 上的 markdown 语法就支持一打(就是很多很多种)的编程语言的语法高亮(但是又不全是真正的语言,像 HTTP headers 的区分),如果你想要看一个完整的语法高亮的语言声明,就看 [highlight.js](http://softwaremaniacs.org/media/soft/highlight/test.html) demo 页面吧

    Inline `code` has `back-ticks around` it.

Inline `code` has `back-ticks around` it.
