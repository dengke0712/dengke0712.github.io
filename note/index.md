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

## 待学习：
- 跑GO和GORM的案例
- 学习文档：
1. https://www.w3cschool.cn/git/
2. https://www.w3cschool.cn/sql/
3. https://www.w3cschool.cn/golang_gin/
4. https://www.w3cschool.cn/linux/
5. https://www.w3cschool.cn/kubernetes/

