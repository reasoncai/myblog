---
title: linux-grep命令详解
date: 2017-05-21 12:30:48
tags: linux命令
categories: linux
---

grep 、sed、awk被称为linux中的"三剑客"。

- grep 更适合单纯的查找或匹配文本
- sed  更适合编辑匹配到的文本
- awk  更适合格式化文本，对文本进行较复杂格式处理

先说说grep命令能做什么？

我们可以使用grep命令在文本中查找指定的字符串，就像你在windows中打开txt文件，使用快捷键 "Ctrl+F" 在文本中查找某个字符串一样，说白了，可以把grep理解成字符查找工具。

grep的全称为： Global search Regular Expression and Print out the line

全称中的"Global search"为全局搜索之意。

全称中的"Regular Expression"表示正则表达式。

所以，从grep的全称中可以了解到，grep是一个可以利用"正则表达式"进行"全局搜索"的工具，grep会在文本文件中按照指定的正则进行全局搜索，并将搜索出的行打印出来。

Unix的grep家族包括grep、egrep和fgrep。egrep和fgrep的命令只跟grep有很小不同。egrep是grep的扩展，支持更多的re元字符， fgrep就是fixed grep或fast grep，它们把所有的字母都看作单词，也就是说，正则表达式中的元字符表示回其自身的字面意义，不再特殊。linux使用GNU版本的grep。它功能更强，可以通过-G、-E、-F命令行选项来使用egrep和fgrep的功能。

当然，不使用正则表达式时也可以使用grep，但是当grep与正则表达式结合在一起时，威力更强大。

grep的常用选项总结如下
- --color=auto 或者 --color：表示对匹配到的文本着色显示
- -i：在搜索的时候忽略大小写
- -n：显示结果所在行号
- -c：统计匹配到的行数，注意，是匹配到的总行数，不是匹配到的次数
- -o：只显示符合条件的字符串，但是不整行显示，每个符合条件的字符串单独显示一行
- -v：输出不带关键字的行（反向查询，反向匹配）
- -w：匹配整个单词，如果是字符串中包含这个单词，则不作匹配
- －l：查询多文件时只输出包含匹配字符的文件名。
- -Ax：在输出的时候包含结果所在行之后的指定行数，这里指之后的x行，A：after
- -Bx：在输出的时候包含结果所在行之前的指定行数，这里指之前的x行，B：before
- -Cx：在输出的时候包含结果所在行之前和之后的指定行数，这里指之前和之后的x行，C：context
- -e：实现多个选项的匹配，逻辑or关系
- -q：静默模式，不输出任何信息，当我们只关心有没有匹配到，却不关心匹配到什么内容时，我们可以使用此命令，然后，使用"echo $?"查看是否匹配到，0表示匹配到，1表示没有匹配到。
- -P：表示使用兼容perl的正则引擎。
- -E：使用扩展正则表达式，而不是基本正则表达式，在使用"-E"选项时，相当于使用egrep。
```
pattern正则表达式主要参数：
\： 忽略正则表达式中特殊字符的原有含义。
^：匹配正则表达式的开始行。
$: 匹配正则表达式的结束行。
\<：从匹配正则表达 式的行开始。
\>：到匹配正则表达式的行结束。
[ ]：单个字符，如[A]即A符合要求 。
[ - ]：范围，如[A-Z]，即A、B、C一直到Z都符合要求 。
。：所有的单个字符。
* ：有字符，长度可以为0。
```
### grep的常用用法 
```bash
[root@www ~]# grep [-acinv] [--color=auto] '搜寻字符串' filename

$ grep ‘test’ d*
显示所有以d开头的文件中包含 test的行。
$ grep ‘test’ aa bb cc
显示在aa，bb，cc文件中匹配test的行。
$ grep ‘[a-z]\{5\}’ aa
显示所有包含每个字符串至少有5个连续小写字符的字符串的行。
$ grep ‘w\(es\)t.*\1′ aa
如果west被匹配，则es就被存储到内存中，并标记为1，然后搜索任意个字符(.*)，这些字符后面紧跟着 另外一个es(\1)，找到就显示该行。如果用egrep或grep -E，就不用”\”号进行转义，直接写成’w(es)t.*\1′就可以了。
```
grep 可以使用 --color=auto 来将关键字部分使用颜色显示。 可以用alias 来处理一下可以在 ~/.bashrc 内加上这行：『alias grep='grep --color=auto'』再以『 source ~/.bashrc 』来立即生效.

#### 根据文件内容递归查找目录
```bash
$ grep ‘test’ *           #在当前目录搜索带'test'行的文件

$ grep -r ‘test’ *        #在当前目录及其子目录下搜索'test'行的文件
$ grep -l -r ‘test’ *     #在当前目录及其子目录下搜索'test'行的文件，但是不显示匹配的行，只显示匹配的文件
```

