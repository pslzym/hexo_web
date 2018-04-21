---
title: vim/gdb操作积累备份
tags: |-

  - Linux操作运维
permalink: vimcao-zuo-ji-lei-bei-fen
id: 16
updated: '2016-11-30 14:41:13'
date: 2016-09-22 11:27:06
---

##vim

shift + G到达文件的底部
gg到达文件的头部
:u用于返回上一个操作

Ctrl+f和Ctrl+b是上下翻页。
Ctrl+u和Ctrl+d是上下翻半页。

n<space> : n如果是20.按下数字后再按空格。光标会向右移动20格。
0  移动到这一行的开头
$  移动到这一行的结尾

:! command 暂时离开vi到指令列模式，执行command的显示结果。

:w [filename] 将编辑的数据存储为另一个文档的数据。

:r [filename] 在编辑的数据中，读入另外一个文档的数据。即将filename这个文档内容加到游标所在行的后面

:n1,n2 w [filename] 将n1到n2行的数据写入filename

区块选择（visual block）：
当我们按下v或者V（ctrl + v）时就进入visual模式
其他操作：
ctrl+v 区块选择，可以用长方形的方式选择资料
y   将反白的地方复制起来
d   将反白的地方删除


    多档案编辑
    vim  XXX1   XXX2
    其他操作：
    :n 编辑下一个档案
    :N 编辑上一个档案
    :files 列出目前这个vim开启的所有档案


    多窗口编辑文件：
    通过vim打开一个文件之后，可以通过相应的命令打开另外一个文件，并且会在同时显示在屏幕上面。
    :sp filename
    其他操作
    ctrl + w + 上
    ctrl + w + 下

替换命令：
最常用： :%s/vivian/sky/g（等同于 :g/vivian/s//sky/g） 替换每一行中所有 vivian 为 sky


    :[range]s/pattern/string/[c,e,g,i]5.1
    range	指的是範圍，1,7 指從第一行至第七行，1,$ 指從第一行至最後一行，也就是整篇文章，也可以 % 代表。還記得嗎？ % 是目前編輯的文章，# 是前一次編輯的文章。
    pattern	就是要被替換掉的字串，可以用 regexp 來表示。
    string	將 pattern 由 string 所取代。
    c	confirm，每次替換前會詢問。
    e	不顯示 error。
    g	globe，不詢問，整行替換。
    i	ignore 不分大小寫。
    http://blog.csdn.net/glorin/article/details/6317098

联想目录的全路径，可以直接: cd /XXX/   即可联想到相关路径  不必退出vim

##gdb
*	gdb开始命令：

```
编译选项：gcc XX.c -o XX -g 
gdb <program>

core文件：
gdb <program> <corefile>

服务程序运行方法：
gdb <program> <PID>   可以指定pid

disass main (查看函数的反汇编代码)
```

*	gdb交互命令：

```
run：简记为 r ，其作用是运行程序，当遇到断点后，程序会在断点处停止运行，等待用户输入下一步的命令。
continue （简写c ）：继续执行，到下一个断点处（或运行结束）
next：（简写 n），单步跟踪程序，当遇到函数调用时，也不进入此函数体；此命令同 step 的主要区别是，step 遇到用户自定义的函数，将步进到函数中去运行，而 next 则直接调用函数，不会进入到函数体内。
step （简写s）：单步调试如果有函数调用，则进入函数；与命令n不同，n是不进入调用的函数的
until：当你厌倦了在一个循环体内单步跟踪时，这个命令可以运行程序直到退出循环体。
until+行号： 运行至某行，不仅仅用来跳出循环
finish： 运行程序，直到当前函数完成返回，并打印函数返回时的堆栈地址和返回值及参数值等信息。
call 函数(参数)：调用程序中可见的函数，并传递“参数”，如：call gdb_test(55)
quit：简记为 q ，退出gdb
```

* gdb设置断点：

```	
break n （简写b n）:在第n行处设置断点
（可以带上代码路径和代码名称： b OAGUPDATE.cpp:578）

b fn1 if a＞b：条件断点设置
break func（break缩写为b）：在函数func()的入口处设置断点，如：break cb_button
delete 断点号n：删除第n个断点
disable 断点号n：暂停第n个断点
enable 断点号n：开启第n个断点
clear 行号n：清除第n行的断点
info b （info breakpoints） ：显示当前程序的断点设置情况
delete breakpoints：清除所有断点：
```

* 查看源代码：

```
list ：简记为 l ，其作用就是列出程序的源代码，默认每次显示10行。
list 行号：将显示当前文件以“行号”为中心的前后10行代码，如：list 12
list 函数名：将显示“函数名”所在函数的源代码，如：list main
list ：不带参数，将接着上一次 list 命令的，输出下边的内容。
```

*	打印表达式

```
print 表达式：简记为 p ，其中“表达式”可以是任何当前正在被测试程序的有效表达式，比如当前正在调试C语言的程序，那么“表达式”可以是任何C语言的有效表达式，包括数字，变量甚至是函数调用。
print a：将显示整数 a 的值
print ++a：将把 a 中的值加1,并显示出来
print name：将显示字符串 name 的值
print gdb_test(22)：将以整数22作为参数调用 gdb_test() 函数
print gdb_test(a)：将以变量 a 作为参数调用 gdb_test() 函数
display 表达式：在单步运行时将非常有用，使用display命令设置一个表达式后，它将在每次单步进行指令后，紧接着输出被设置的表达式及值。如： display a
watch 表达式：设置一个监视点，一旦被监视的“表达式”的值改变，gdb将强行终止正在被调试的程序。如： watch a
whatis ：查询变量或函数
info function： 查询函数
扩展info locals： 显示当前堆栈页的所有变量
```

* 查询运行信息
	
```
where/bt ：当前运行的堆栈列表；
bt backtrace 显示当前调用堆栈
up/down 改变堆栈显示的深度
set args 参数:指定运行时的参数
show args：查看设置好的参数
info program： 来查看程序的是否在运行，进程号，被暂停的原因。
```

*	分割窗口

```
layout：用于分割窗口，可以一边查看代码，一边测试：
layout src：显示源代码窗口
layout asm：显示反汇编窗口
layout regs：显示源代码/反汇编和CPU寄存器窗口
layout split：显示源代码和反汇编窗口
Ctrl + L：刷新窗口
refresh : 刷新窗口
ctrl + x + a 退出layout
info win显示窗口的大小
focus cmd/src/asm/regs/next/prev 切换当前窗口
```

* 多进程调试

```
follow-fork-mode  detach-on-fork   说明
parent               on       只调试主进程（GDB默认）
child                on       只调试子进程
parent               off      同时调试两个进程，gdb跟主进程，子进程block在fork位置
child                off      同时调试两个进程，gdb跟子进程，主进程block在fork位置

设置方法：set follow-fork-mode [parent|child]   set detach-on-fork [on|off]

info inferiors：查询正在调试的进程
inferior <infer number> ：  切换调试的进程
```

* gdb多线程调试

```
info thread 查看当前进程的线程。
thread <ID> 切换调试的线程为指定ID的线程。
break file.c:100 thread all  在file.c文件第100行处为所有经过这里的线程设置断点。
set scheduler-locking off|on|step，这个是问得最多的。在使用step或者continue命令调试当前被调试线程的时候，其他线程也是同时执行的，怎么只让被调试程序执行呢？通过这个命令就可以实现这个需求。
off 不锁定任何线程，也就是所有线程都执行，这是默认值。
on 只有当前被调试程序会执行。
step 在单步的时候，除了next过一个函数的情况(熟悉情况的人可能知道，这其实是一个设置断点然后continue的行为)以外，只有当前线程会执行。
```

