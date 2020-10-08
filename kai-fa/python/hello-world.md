# Hello,world!

在开始编写python前需要了解一些基本的概念

* 不需要预先定义变量或参数
* python对于语法格式是强制规范的
  * 代码块以缩进（默认为四个空格）的方式划分而不是花括号
  * 语句中的符号间隔为一个空格

python的运行方式分为三种：命令行模式，交互模式与脚本模式。

### 命令行模式

命令行模式可以像执行shell脚本一样执行一行或多行的python语句。通过如下命令执行

```text
python3 -c 'print("Hello,world!")'
```

参数`-c`表示使用命令行模式执行python语句

### 交互模式

通过命令python3不携带任何参数可进入交互模式，在交互模式中可以直接执行各种python语句，每一个完整语句的执行都以`>>>`开始，退出交互模式使用方法`exit()`。

```text
$ python3
Python 3.8.2 (default, Mar  9 2020, 16:57:08) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> print("Hello,world!")
Hello,world!
>>> exit()
```

在使用if语句时会自动进入多行输入模式，以`...`表示多行输入

```text
>>> flag = True
>>> if flag:
...     print("Hello,world!")
... 
Hello,world!
```

在进行运算时，如果没有将结果赋值给变量，则会输出运算结果

```text
>>> 2 + 3
5
```

将结果赋值给变量之后可以在后续的运算中使用

```text
>>> a = 3 - 2
>>> b = 4 + 5
>>> a + b
10
```

 在交互模式下，上一次打印出来的结果被赋值给内置变量 `_`

```text
>>> 3 + 3
>>> 4 + _
10
```

### 脚本模式

可以将python语句写入到文件中做为脚本文件，通过python解析该脚本并执行其中的语句

```text
$ cat >> hello_world.py << EOF
> print("Hello,world!")
EOF

$ python3 hello_world.py
Hello,world!
```

