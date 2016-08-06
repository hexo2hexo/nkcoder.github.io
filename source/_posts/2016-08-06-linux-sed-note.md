title: Linux命令之sed
categories: Backend
date: 2016-08-06 14:23:42
tags: [linux, sed]
---

sed用于处理文本，逐行处理，不需交互， 数据源可以是标准输入流，也可以是文件；处理后的内容默认是输出到标准输出流，
可以重定向，也可以原地修改文件，如果是原地修改文件，支持备份；

sed命令的语法：

    sed [options] commands [file-to-edit]

下面以annoying.txt文件作为示例数据源， 文件内容为：

    root@a01:~/junk# echo "this is the song that never ends
    > yes, it goes on and on, my friend
    > some people started singing it
    > not knowing what it was
    > and they'll continue singing it forever
    > just because..." > annoying.txt

<!-- more -->

# 1. 基本用法

## 1.1 读取文件内容

如果`commands`为空，表示对每一行没有做任何处理，相当于读取文件的内容（因为sed默认会将读到的内容打印出来），如：

    root@a01:~/junk# sed '' annoying.txt
    this is the song that never ends
    yes, it goes on and on, my friend
    some people started singing it
    not knowing what it was
    and they'll continue singing it forever
    just because...

这与执行：`# cat annoying.txt | sed ''`的输出是一样的。

## 1.2 打印命令p

p命令将读到的内容直接打印出来， 如：

    root@a01:~/junk# sed 'p' annoying.txt
    this is the song that never ends
    this is the song that never ends
    yes, it goes on and on, my friend
    yes, it goes on and on, my friend
    some people started singing it
    some people started singing it
    not knowing what it was
    not knowing what it was
    and they'll continue singing it forever
    and they'll continue singing it forever
    just because...
    just because...

上面每一行都会打印两遍，是因为sed默认会将读到的内容输出，使用`-n`选项可以禁用默认的输出，如：

    root@a01:~/junk# sed -n 'p' annoying.txt
    this is the song that never ends
    yes, it goes on and on, my friend
    some people started singing it
    not knowing what it was
    and they'll continue singing it forever
    just because...

范围限制：可以在p命令前增加范围限制，数字表示具体的行数，+数字表示增量，$表示最后一行，~表示间隔的行数，如：

打印第1行：

    root@a01:~/junk# sed -n '1p' annoying.txt
    this is the song that never ends

打印第1行到第2行：

    root@a01:~/junk# sed -n '1,2p' annoying.txt
    this is the song that never ends
    yes, it goes on and on, my friend

打印第3行到最后一行：

    root@a01:~/junk# sed -n '3,$p' annoying.txt
    some people started singing it
    not knowing what it was
    and they'll continue singing it forever
    just because...

打印第1行到后面的2行，即前3行:

    root@a01:~/junk# sed -n '1,+2p' annoying.txt
    this is the song that never ends
    yes, it goes on and on, my friend
    some people started singing it

每隔2行打印，即打印奇数行：

    root@a01:~/junk# sed -n '1~2p' annoying.txt
    this is the song that never ends
    some people started singing it
    and they'll continue singing it forever

## 1.3 删除命令d

删除命令d与打印命令p的用法基本类似，只需要将p换成d即可，如：

    root@a01:~/junk# sed '1~2d' annoying.txt
    yes, it goes on and on, my friend
    not knowing what it was
    just because...

    root@a01:~/junk# sed '1,3d' annoying.txt
    not knowing what it was
    and they'll continue singing it forever
    just because...

使用删除命令，没有被删的行会被打印出来，但是原文件是不被影响的。可以使用`-i`选项，表示在原文件上直接修改，
但是这样是比较危险的，最好先备份，而`-i`选项支持备份，`-i`后面的参数值表示备份文件的后缀，如：

    root@a01:~/junk# sed -i '1d' annoying.txt

    root@a01:~/junk# sed -i.bak '1d' annoying.txt
    root@a01:~/junk# ls
    annoying.txt  annoying.txt.bak

    root@a01:~/junk# cat annoying.txt
    some people started singing it
    not knowing what it was
    and they'll continue singing it forever
    just because...

    root@a01:~/junk# cat annoying.txt.bak
    this is the song that never ends
    yes, it goes on and on, my friend
    some people started singing it
    not knowing what it was
    and they'll continue singing it forever
    just because...

## 1.4 替换命令s

    's/old_word/new_word/'

s是替换命令，/是默认的分隔符，也可以使用其它字符作为分隔符（紧跟s之后），注意末尾的分隔符不能省略，如：

    root@a01:~/junk# echo http://www.thegeekstuff.com/2009/09/unix-sed | sed 's_2009/09_2016/07_'
    http://www.thegeekstuff.com/2016/07/unix-sed

s命令默认仅替换每一行出现的第1个匹配，如：

    root@a01:~/junk# sed 's/on/forward/' annoying.txt
    this is the sforwardg that never ends
    yes, it goes forward and on, my friend
    some people started singing it
    not knowing what it was
    and they'll cforwardtinue singing it forever
    just because...

也可以仅替换每行的第n个匹配，如：

    root@a01:~/junk# sed 's/on/forward/2' annoying.txt
    this is the song that never ends
    yes, it goes on and forward, my friend
    some people started singing it
    not knowing what it was
    and they'll continue singing it forever
    just because...

全局替换使用g参数：

    root@a01:~/junk# sed 's/on/forward/g' annoying.txt
    this is the sforwardg that never ends
    yes, it goes forward and forward, my friend
    some people started singing it
    not knowing what it was
    and they'll cforwardtinue singing it forever
    just because...

如果仅替换独立的单词，不替换单词的部分，使用\b限制边界，如：

    root@a01:~/junk# sed 's/\bon\b/forward/g' annoying.txt
    this is the song that never ends
    yes, it goes forward and forward, my friend
    some people started singing it
    not knowing what it was
    and they'll continue singing it forever
    just because...

限制操作的行数，如仅全局替换前3行（&表示引用匹配到的部分）：

    root@a01:~/junk# sed '1,2s/^.*/  &/' annoying.txt
      this is the song that never ends
      yes, it goes on and on, my friend
    some people started singing it
    not knowing what it was
    and they'll continue singing it forever
    just because...

可以使用-n选项和p参数，打印仅发生替换的内容，如：

    root@a01:~/junk# sed -n 's/on/forward/gp' annoying.txt
    this is the sforwardg that never ends
    yes, it goes forward and forward, my friend
    and they'll cforwardtinue singing it forever

可见，在s命令的最后，可以通过参数影响替换的行为，如i表示忽略大小写：

    root@a01:~/junk# sed -n 's/on/forward/gpi' annoying.txt
    this is the sforwardg that never ends
    yes, it goes forward and forward, my friend
    and they'll cforwardtinue singing it forever

引用被匹配的部分：如果只有一个分组，则使用&比较方便，表示引用被匹配的部分，如：

    root@a01:~/junk# sed 's/.*/| &/' annoying.txt
    | this is the song that never ends
    | yes, it goes on and on, my friend
    | some people started singing it
    | not knowing what it was
    | and they'll continue singing it forever
    | just because...

如果有多个分组，首先在`old_word`里通过括号()分组，然后在`new_word`里通过数字需要去引号（分组的括号以及引用的数字都需要通过\转义）,
数字和前面的分组一一对应。
比如我们要反转每一行的前两个单词：

    root@a01:~/junk# sed 's/\([0-9a-zA-Z][0-9a-zA-Z]*\) \([0-9a-zA-Z][0-9a-zA-Z]*\)/\2 \1/' annoying.txt
    is this the song that never ends
    yes, goes it on and on, my friend
    people some started singing it
    knowing not what it was
    they and'll continue singing it forever
    because just...

考虑到字符后的改进版：

    root@a01:~/junk# sed 's/\([^ ][^ ]*\) \([^ ][^ ]*\)/\2 \1/' annoying.txt
    is this the song that never ends
    it yes, goes on and on, my friend
    people some started singing it
    knowing not what it was
    they'll and continue singing it forever
    because... just

# 2. 中级用法

## 2.1 执行多条处理命令：-e选项

可以通过管道|，如：

    root@a01:~/junk# sed 's/and/\&/' annoying.txt | sed 's/people/horses/'
    this is the song that never ends
    yes, it goes on & on, my friend
    some horses started singing it
    not knowing what it was
    & they'll continue singing it forever
    just because...

但是因为需要多次调用sed命令，因此效率不好。推荐使用sed的`-e`选项来执行多条命令（只有一条命令时，-e选项不是必须的），如：

    root@a01:~/junk# sed -e 's/and/\&/' -e 's/people/horses/' annoying.txt
    this is the song that never ends
    yes, it goes on & on, my friend
    some horses started singing it
    not knowing what it was
    & they'll continue singing it forever
    just because...

还有一种方式，通过分号(;)将命令分割，如：

    root@a01:~/junk# sed 's/and/\&/;s/people/horses/' annoying.txt
    this is the song that never ends
    yes, it goes on & on, my friend
    some horses started singing it
    not knowing what it was
    & they'll continue singing it forever
    just because...

## 2.2 sed脚本文件：-f选项

可以将sed命令放到文件里，然后执行这个脚本文件，语法为：

    $ sed -f script-file file-to-edit

比如：

    root@a01:~/junk# echo 's/and/\&/
    > s/people/horses/' >> sed-demo

    root@a01:~/junk# sed -f sed-demo annoying.txt
    this is the song that never ends
    yes, it goes on & on, my friend
    some horses started singing it
    not knowing what it was
    & they'll continue singing it forever
    just because...

## 2.3 范围限制

可以通过正则表达式限制处理的范围（即行数范围）。

比如仅处理包含singing这个单词的行：

    root@a01:~/junk# sed '/singing/s/it/& loudly/' annoying.txt
    this is the song that never ends
    yes, it goes on and on, my friend
    some people started singing it loudly
    not knowing what it was
    and they'll continue singing it loudly forever
    just because...

删除从some开头到not开头的中间所有行：

    root@a01:~/junk# sed '/^some/,/^not/d' annoying.txt
    this is the song that never ends
    yes, it goes on and on, my friend
    and they'll continue singing it forever
    just because...

命令前面使用!表示对命令取反，比如对上一条命令取反：

    root@a01:~/junk# sed '/^some/,/^not/!d' annoying.txt
    some people started singing it
    not knowing what it was

## 参考

- [The Basics of Using the Sed Stream Editor to Manipulate Text in Linux](https://www.digitalocean.com/community/tutorials/the-basics-of-using-the-sed-stream-editor-to-manipulate-text-in-linux)
- [Intermediate Sed: Manipulating Streams of Text in a Linux Environment](https://www.digitalocean.com/community/tutorials/intermediate-sed-manipulating-streams-of-text-in-a-linux-environment)

