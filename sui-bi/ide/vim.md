# vim

## Q&A

### 如何修改文件格式？

在windows下编辑过的文件传输到linux上后执行时出现类似错误

```text
/bin/bash^M: bad interpreter: No such file or directory
```

可以通过`:set ff=unix`保存为linux所能识别的格式，可以通过`:set ff=dos`再重新保存回windows所能识别的格式



