---
title: python 知识积累
tags: |-

  - python
permalink: python-zhi-shi-ji-lei
id: 15
updated: '2016-09-22 11:15:05'
date: 2016-09-22 11:05:07
---

##python 语法积累

Python 数的数据类型：

     type函数可以显示变量类型

       字符串：
               单引号和双引号使用完全相同
               使用'''  或者 """可以指定一个多行的字符串。
               转移符 '\'
               自然字符串， 通过在字符串前加r或R。 如 r"this is a line with \n" 则\n会显示，并不是换行。
               python允许处理unicode字符串，加前缀u或U， 如 u"this is an unicode string"。
               字符串是不可变的。
               按字面意义级联字符串，如"this " "is " "string"会被自动转换为this is string。
       对象：
             Python程序中用到的任何东西都将成为对象
         
       逻辑行和物理行：
	           物理行是我们在编写程序时看到的，逻辑行则是python看到的。
	           python 中的分号；标识一个逻辑行的结束，但是实际中一般每个物理行只写一个逻辑行，可以避免使用分号。
	           
	           s = "peter is \
	           writing this article"
	           
	           上面\的使用被称为‘明确的行连接’
                       
                       
缩进：
      空白在python是非常重要的，行首的空白是最重要的，又称为缩进。行首的空白（空格和制表符）用来决定逻辑行的缩进层次，从而决定语句
分组：这意味着同一层次的语句必须有相同的缩进，每一组这样的语句称为一个块。注意：不要混合使用空格和制表符来缩进，因为在跨越不同的平
台时无法正常工作。


python ：控制台输出 
       print "abc"
       print "abc%s" % "d"      #abcd
       
       
控制流：
        i = 10
n = int(raw_input("enter a number:"))

if n == i:
    print "equal"
elif n < i:
    print "lower"
else:
    print "higher"
    

while 语句：

while True:
      pass
else:
     pass
     
for语句：
        
for i in range(0,5):
    print i 
else:
     pass

注：当for循环结束后执行else语句；

　　　　range(a, b)返回一个序列，从a开始到b为止，但不包括b，range默认步长为1，可以指定步长，range(0,10,2)；

　　4. break语句

　　　　终止循环语句，如果从for或while中终止，任何对应循环的else将不执行。

　　5. continue语句

　　　　continue语句用来调过当前循环的剩余语句，然后继续下一轮循环。


函数：
      函数通过def定义。def关键字后面跟函数的标识符名称，然后跟一对圆括号，括号里面包含一些变量名，最后以冒号结尾
      
def sumOf(a, b):
    return a + b
    
    局部变量：在函数内定义的变量与函数外具有相同名称的其他变量没有任何关系，即变量名称对于函数来说是局部的。这称为变量的作用域。

　　　　global语句， 为定义在函数外的变量赋值时使用global语句。

def func():
    global x
    print "x is ", x
    x = 1

x = 3
func()
print x


3. 默认参数

　　　　通过使用默认参数可以使函数的一些参数是‘可选的’。

def say(msg, times =  1):
    print msg * times

say("peter")
say("peter", 3)


4.关键参数：

def func(a, b=2, c=3):
    print "a is %s, b is %s, c is %s" % (a, b, c)

func(1) #a is 1, b is 2, c is 3
func(1, 5) #a is 1, b is 5, c is 3
func(1, c = 10) #a is 1, b is 2, c is 10
func(c = 20, a = 30) #a is 30, b is 2, c is 20


5.模块
      模块就是一个包含了所有你定义的函数和变量的文件，模块必须以.py为扩展名。模块可以从其他程序中‘输入’(import)以便利用它的功能。

　　  在python程序中导入其他模块使用'import', 所导入的模块必须在sys.path所列的目录中，因为sys.path第一个字符串是空串''即当前目录，所以程序中可导入当前目录的模块。

 　　 1. 字节编译的.pyc文件

　　　　导入模块比较费时，python做了优化，以便导入模块更快些。一种方法是创建字节编译的文件，这些文件以.pyc为扩展名。

　　　　pyc是一种二进制文件，是py文件经编译后产生的一种byte code，而且是跨平台的（平台无关）字节码，是有python虚拟机执行的，类似于

        java或.net虚拟机的概念。pyc的内容，是跟python的版本相关的，不同版本编译后的pyc文件是不同的。
        
      2. form ....import ...
      
      如果想直接使用其他模块的变量或其他，而不加'模块名+.'前缀，可以使用from .. import。

　　　　例如想直接使用sys的argv，from sys import argv 或 from sys import *

六.数据结构

python有三种内建的数据结构：列表、元组和字典。

1.列表
      list是处理一组有序项目的数据结构，列表是可变的数据结构。列表的项目包含在方括号[]中，
      eg: [1, 2, 3]， 空列表[]。判断列表中是否包含某项可以使用in， 比如 l = [1, 2, 3];
      print 1 in l; #True；支持索引和切片操作；索引时若超出范围，则IndexError；
      使用函数len()查看长度；使用del可以删除列表中的项，eg: del l[0] # 如果超出范围，
      则IndexError



      append(value)
      
      count(value) 
      
      extend(list2)
————————————————————————————————————————————
list  []  和tuple  ()的区别
dict  {}  和set   输入必须是一个list ([])   重复的元素自动被过滤   的区别

set可以看成数学意义上的无序和无重复元素的集合，因此，两个set可以做数学意义上的交集、并集等操作：





-------------------------------------------
	http://python.usyiyi.cn/
	http://www.pythondoc.com/

repr()显示函数   把变量等显示为字符串

json.dumps -->把变量等转换为json格式
json.loads  -->json转变量等


-------------------------------------------
python 包结构和导入相关

http://fengwy.blog.chinaunix.net/uid-26602509-id-3499026.html

------------------------------------------
常用函数：


-----------------------------------------
面向对象编程：
http://www.cnblogs.com/dolphin0520/archive/2013/03/29/2986924.html

-----------------------------------------
去掉字符串的换行符：

	for line in file.readlines():
   		line=line.strip('\n')

----------------------------------------
python 修饰符@

http://www.cnblogs.com/xupeizhi/archive/2013/02/07/2908600.html

---------------------------------------


--------------------------------------
python   virtualenv 环境：
http://www.cnblogs.com/kevin922/p/4315780.html

-------------------------------------
数字与字符串相互转换：

string模块里有
a="12345"
import string
string.atoi(a)
12345
b="123.678"
string.atof(b)
123.678


字符串str转换成int: int_value = int(str_value)
int转换成字符串str: str_value = str(int_value)
int -> unicode: unicode(int_value)
unicode -> int: int(unicode_value)
str -> unicode: unicode(str_value)
unicode -> str: str(unicode_value)
int -> str: str(int_value)
str -> int: int(str_value)
