# 类型

## 变量

python中使用变量是不需要提前定义的，在使用时直接赋值即可，其类型也会自动推断。且变量可以服务任何类型值。如下的赋值方式是合法的：

```text
>>> a = 1
>>> a = True
>>> a = "abc"
```

{% hint style="info" %}
如果使用一个未赋值的变量会引发错误
{% endhint %}

```text
>>> n
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'n' is not defined
```



