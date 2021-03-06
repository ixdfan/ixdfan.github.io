---
layout: post
title: shell基础
categories:
- shell
---

shell命令的种类：

	1.内建命令
	2.shell函数
	3.外部函数

========================================================================

内建命令是shell程序本身包含的命令。这些命令集成在shell解析器中。

例如：所有shell解析器中都包含cd内建命令来改变工作目录内建命令的存在是为了改变shell本身的属性设置。

**在执行内建命令时，没有进程的创建和消亡；**

另一部分内建命令则是IO命令，例如echo命令；

========================================================================

shell函数式一系列函数代码，以shell语言写成，它可以像其他命令一样被引用。

========================================================================

外部命令是独立于shell的可执行程序，例如find、grep、echo.sh。命令行shell在执行外部命令时，会创建一个当前shell的复制进程来执行，在执行过程中，存在进程的创建和消亡。
外部命令的执行如下：

	(1).调用POSIX系统fork函数接口，创建一个命令行shell进程的子进程。
	(2).在子进程的运行环境中，查找外部命令在Linux文件系统中的位置。如果外部命令给出了完全路径，则跳过这一步。
	(3).在子进程里，以新进程取代shell拷贝并执行(exec)，此时父进程进入休眠，等待子进程执行完毕。
	(4).子进程执行完毕后，父进程接着从终端读取下一条命令。

========================================================================

##### 注意：
1. 子进程创建初期和父进程一模一样，但是子进程不能改变父进程参数变量
2. 只有内建命令才能够改变命令行shell的属性设置(环境变量)

========================================================================

实例：echo.sh

    
    #! /bin/sh
    cd /tmp
    echo "hello world!"


执行结果：

    
    [root@localhost 01]# pwd
    /root/workspace/shell/01
    [root@localhost 01]# chmod +x echo.sh
    [root@localhost 01]# ./echo.sh
    hello world!
    [root@localhost 01]# pwd
    /root/workspace/shell/01
    [root@localhost 01]#


会发现路径并没有改变

========================================================================
    
    [root@localhost 01]# pwd
    /root/workspace/shell/01
    [root@localhost 01]# source echo.sh
    hello world!
    [root@localhost tmp]# pwd
    /tmp


现在目录发生了改变

========================================================================

执行过程：

	(1)父进程接收到命令"./echo.sh"时，发现不是内建命令于是创建一个和自己一样的shell进程(子进程)来执行外命令。
	(2)在shell子进程中使用/bin/sh来取代自己，shell进程设置自己的运行环境变量其中包括$PWD(标识当前工作目录)。
	(3)sh进程一次执行内建命令cd和echo，在此进程中，sh进程(子进程)的环境变量$PWD被cd命令改变。
	注意：父进程的环境变量并没有改变
	(4)sh子进程执行完毕，消失。一直在等待的父进程醒来继续接受命令
	所以在第一次运行的时候目录没有改变，因为父进程的当前目录(环境变量)无法被子进程改变！

##### 注意：
在使用source执行shell脚本时，不会创建子进程，而是在父进程中直接进行！

========================================================================

source语法:

	source file
	. file

表示使用shell进程本身执行脚本文件。source命令也称为"点命令"通常用于重新执行刚刚修改的初始化文件，使之立即生效。

========================================================================

	[root@localhost 01]# var=123
	[root@localhost 01]# echo "$var"
	123
	[root@localhost 01]# echo '$var'
	$var

可以看到单引号中的var并没有被替换成为变量值123，变量替换被禁止了，而双引号中的$var发生了变量替换；

##### 注意：
单引号为全引用(强引用)，双引号为弱引用。

=================================================================================================

shell中变量允许为空值(NULL),就是不含有任何字符。但是在算术运算中这个未初始化的变量不一定就是0。

	[root@localhost 01]# echo "$uninit" //为空

	[root@localhost 01]# let "uninit+=5"
	[root@localhost 01]# echo "$uninit"
	5
虽然结果是5，但是要谨慎使用

=================================================================================================

shell的局部变量与全局变量

##### 注意：
* 局部变量必须以local声明，否则就是变量是属于某一代码块的，它也是全局可见的
* 环境变量时全局变量的一种，全局变量在全局范围内可见，声明全局变量不需要任何修饰

示例代码

    
    #!/bin/sh
    
    num=123
    fun1()
    {
            num=321
            echo $num
    }
    
    fun2()
    {
            local num=456
            echo $num
    }
    echo $num
    fun1
    echo $num
    fun2
    echo $num


输出结果

	[root@localhost 01]# ./local.sh
	123 //初始值
	321 //fun1内被改变
	321 //fun1中影响了全局变量
	456 //fun2中的num
	321 //fun2中没有对全局变量产生影响

=================================================================================================
**export主要用于设置或显示环境变量**

export的语法

	export [-fnp] [变量名]=[变量设置值]
	-f 代表[变量名称]中的函数名称
	-n 删除指定的变量，实际上变量并没有被删除，只是不会输出到后续指令的执行环境中
	-p 列出所有的shell赋予程序的环境变量
##### 注意：
* export仅仅修改当前shell进程的环境变量，若讲export命令置于脚本中被调用执行，则export对父进程的环境变量是没有改变的
* 使用source执行脚本的时候没有子进程产生，所以脚本中export命令将会影响父进程的环境
* export设置的环境变量的有效期仅仅维持到当前进程消亡为止，所以下次登录时环境变量无效，想要永久的保存export的环境变量，可以将export置于shell登录时的启动文件中。

	export PATH=/bin:/usr/bin:/usr/local/bin

=================================================================================================

##### shell中常见的环境变量

	HOME 用户的专属目录

	PATH 外部命令的搜索路径

	LOGNAME 当前用户的登录名

	SHELL 当前用户使用的shell类型

	MAIL 当前用户的邮件存放目录

=================================================================================================
