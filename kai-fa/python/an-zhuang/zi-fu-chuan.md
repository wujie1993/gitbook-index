# 字符串

字符串表达式通过`'...'`或`"..."`表示。两者可以互相嵌套，但不能多次使用。这两者间的唯一区别是单引号可以在不使用转义符号的时候嵌套双引号。

```text
>>> "mike"
'mike'
>>> 'green river shcool'
'green river shcool'
>>> 'hello,'mike!''
'hello,"mike!"'
>>> 'hello,'mike!''
  File "<stdin>", line 1
    'hello,'mike!''
            ^
SyntaxError: invalid syntax
```

### 字符串转义

使用`/`符号可以表示转义字符。

```text
>>> words = "Hello!\nHi!\tHow are you?"
>>> print(words)
Hello!
Hi!	How are you?
```

如果不想要让`/`进行转义，可以在字符串表达式的引号前添加`r`表示使用原始字符串。

```text
>>> print("hello\nhi")
hello
hi
>>> print(r"hello\nhi")
hello\nhi
```

### 多行字符串

对于一些较长的文本字符串，可能会存在多行，用过多转义字符会影响代码的可读性。可以使用`"""..."""`或`'''...'''`表示多行文本字符串，在其中直接换行输入而不需要使用到转义换行符。如果要取消自动换行可以在对应行末加\符号，表示将当前行与下一行间的换行去除。

```text
>>> print("""\
... Usage: thingy [OPTIONS]
...      -h                        Display this usage message
...      -H hostname               Hostname to connect to
... """)
Usage: thingy [OPTIONS]
     -h                        Display this usage message
     -H hostname               Hostname to connect to

```

### 下标索引

在字符串末尾使用符号`[]`可以对字符串中对应下标位置的字符进行索引，索引位置从0开始

```text
>>> 'abc'[1]
'b'
>>> str = 'abc'
>>> str[0]
'a'
```

正常情况下索引一遍从左往右，如果需要从右往左索引，可以使用负数索引下标，这种情况下索引以-1做为起始位置

```text
>>> 'python'[-1]
'n'
```

无论是从哪个方向索引，下标均不能超过字符串的长度，否则会引发下标越界错误

```text
>>> 'python'[-7]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: string index out of range
```

### 字符串切片

可以使用切片表示字符串的一段区间，用法：`string[start:end]`，意思为取字符串`string`中下标`start`到`end`之前（不包含`end`）这段区间，其中`end`必须大于`start`。

```text
>>> 'python'[2:4]
'th'
>>> 'python'[-3:-1]
'ho'
```

如果需要表示从头开始或从尾开始取字符串切片，可以省略`start`或`end`

```text
>>> 'python'[:3]
'pyt'
>>> 'python'[2:]
'thon'
>>> 'python'[-3:]
'hon'
>>> 'python'[:-2]
'pyth'
```

与下标索引不同的是，切片中的下标越界不会引发错误，而是自动忽略越界部分

```text
>>> 'python'[3:10]
'hon'
```

### 字符串连接

可以使用符号`+`（建议）或者是空格连接两个字符串

```text
>>> 'abc' + 'def'
'abcdef'
>>> 'abc' 'def'
'abcdef'
```

需要注意的是空格不能用于连接字符串变量，只适用于字面值表示的字符串，以下用法是错误的

```text
>>> str = 'abc'
>>> str 'def'
  File "<stdin>", line 1
    str 'def'
        ^
SyntaxError: invalid syntax
```

如果需要对字符串做多次重复连接，可以使用符号`*`

```text
>>> 'abc' * 3 
'abcabcabc'
```

### 截取字符串

字符串是不可修改的，对于字符串的内容修改事实上需要新建字符串，通过字符串的切片操作完成

例子：替换字符串中下标3的元素

```text
>>> str = 'python'
>>> str[:3] + 'a' + str[4:]
'pytaon'
```

与此类似，删除元素也可以通过切片完成

例子：截去字符串中下标3到4的元素

```text
>>> str = 'python'
>>> str[:3] + str[5:]
'pytn'
```

