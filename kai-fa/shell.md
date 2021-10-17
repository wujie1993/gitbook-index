# Shell

### 循环

例子：循环执行指令10次

```
for i in {1..10}; do echo "Hello, World"; done
```

### 分支

例子：使用 if 进行分支控制

```
if [ $score >=80  ]; then
    echo "great"
elif [ $score >= 60 ]; then
    echo "good"
else
    echo "bad"
fi
```

例子：使用 switch 进行分支控制

```
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
