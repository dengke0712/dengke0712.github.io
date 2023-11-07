# Notes


<!--more-->

## 笔记

GORM文件初始化
``` 
go mod init module_name
go mod tidy
go mod vendor
```

去掉.idea
``` 
git rm -rf .idea --cached
``` 
去掉.DS
``` 
touch ~/.gitignore_global
open ~/.gitignore_global

在 .gitignore_global文件中输入
*~
.DS_Store
._.DS_Store
._*
可以去掉.DS

git config --global core.excludesfile ~/.gitignore_global
```


