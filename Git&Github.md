# Git



## Git删除仓库中的文件或文件夹

从仓库中删除文件或文件夹,而本地保留

使用`git rm`命令

### 文件

```shell
git rm readme.md

git commit -m "remove readme.md"
```

### 文件夹

```shell
git rm -r --cached venv

git commit -m 'remove venv' 
```

