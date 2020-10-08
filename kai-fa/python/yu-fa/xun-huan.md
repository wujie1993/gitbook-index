# 循环

### while

while语句用于在满足某个条件的情况下循环执行语句，语法格式如下：

```text
while {{ 条件语句 }}:
    ...
```

例子：从1累加到100

```text
>>> i = 1
>>> sum = 0
>>> while i <= 100:
...     sum += i
...     i += 1
...
>>> print(sum)
5050
```

### for

for语句用于对可遍历的类型进行迭代遍历，在每一轮的循环前，按存储的顺序从集合中取出一个元素，之后执行循环体中的部分，语法格式如下：

```text
for {{ 元素索引 }},{{ 元素值 }} in {{ 集合 }}:
    ...
```

在不需要使用到元素索引的情况下可以忽略元素索引

```text
for {{ 元素值 }} in {{ 集合 }}:
    ...
```

可用于遍历的类型包括：字符串，列表，元组，集合和字典

例子：遍历输出列表中每个用户的信息

```text
>>> members = [
...     {'name': "mike", "age": 18, "gender": 1},
...     {"name": "mary", "age": 24, "gender": 0},
...     {"name": "bob", "age": 65, "gender": 1}
... ]
>>> for person in members:
...     for k, v in person.items():
...         print(k, v)
...
name mike
age 18
gender 1
name mary
age 24
gender 0
name bob
age 65
gender 1
```

当我们需要通过循环获取列表的元素下标及元素值时，可以使用内置的方法`range()`，这个方法中支持多种参数传递方式。当只传一个参数时，如range\(5\)，表示一个从0到5（不包括5）且步长为1的等差数列。当传递两个参数时，如range\(3, 12\)，表示一个从3到12（不包括12）且步长为1的等差数列。当传递三个参数时，如range\(3, 13, 2\)，表示一个从3到13（不包括13）且步长为2的等差数列。

我们通过len\(\)方法获取原列表的长度，并做为range\(\)的参数即可获得一个与原列表对应的元素下标列表，使用for循环遍历该元素下标列表可获得列表的下标值，通过列表索引该下标值即可获取元素值。

例子：根据列表下标遍历每个用户的信息

```text
>>> members = [
...     {'name': "mike", "age": 18, "gender": 1},
...     {"name": "mary", "age": 24, "gender": 0},
...     {"name": "bob", "age": 65, "gender": 1}
... ]
>>> for index in range(len(members)):
...     for k, v in members[index].items():
...         print(index, k, v)
...
0 name mike
0 age 18
0 gender 1
1 name mary
1 age 24
1 gender 0
2 name bob
2 age 65
2 gender 1
```

需要注意的是`range()`生成的是一个可遍历对象，而不是一个列表

```text
>>> print(range(5))
range(0, 5)
```

需要将其转化为一个实际的列表时，可以使用内置方法`list()`

```text
>>> list(range(2, 14, 2))
[2, 4, 6, 8, 10, 12]
```

要累加`range()`中的内容时，可以使用内置方法`sum()`

例子：使用sum\(\)从1加到100

```text
>>> sum(range(1, 101))
5050
```



