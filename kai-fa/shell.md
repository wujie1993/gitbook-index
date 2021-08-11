# Shell

### 循环

例子：循环执行指令10次

```text
for i in {1..10}; do echo "Hello, World"; done
```

### 分支

例子：根据参数值的不同输出不同的内容

```text
case $msg in
    "hello")
        echo "hi"
        ;;
    "goodbye")
        echo "bye"
        ;;
    *)
        echo "i have no idea for what you means"
        ;;
esac
```

