# Git

### 初始化仓库

```text
git init
```

### 配置用户名和邮箱

```text
git config user.username "yourname"
git config user.email "your@email.com"
```

如果需要使配置生效，可附加参数`--global`

## 实用技巧

### 将多次提交进行合并



### 撤销上回的提交

1. `git reset commitId`，\(注：不要带--hard\)到上个版本  
2. `git stash`，暂存修改  
3. `git push --force`, 强制push,远程的最新的一次commit被删除  
4. `git stash pop`，释放暂存的修改，开始修改代码  
5. `git add .` -&gt; `git commit -m "massage"` -&gt; `git push`

如果是仅修改commit描述

1. `git commit --amend`\(注：修改完成后`Esc+wq+Enter`保存退出\)  
2. `git push --force`

