```git
git clone 地址   从远程仓库获取项目
```

会在当前面目录创建一个以仓库为名字的文件夹



```
git status 查看当前git状态
git branch -m oldname newname 更改分支名
git checkout -b dev(本地分支名) origin/develop(远程分支名) 拉取指定的分支，注意最好写同样的名字，否则push会提示名字不同
git branch -a 显示远程所有分支
git commit时需要先设置邮箱和用户名，有提示
HEAD在 git 中，它是一个指向你正在工作中的本地分支的指针，可以将 HEAD 想象为当前分支的别名
git branch 分支名 创建分支


## iss1解决后，把修改合并会master，并删除iss1分支
$ git checkout master
$ git merge iss1
$ git branch -d iss1  删除本地分支
$ git push
## 删除远程分支
$ git push origin :iss1


当我们push时，可能需要先pull再push，因为别人可能已经对该分支进行了修改，
同时当pull时很可能对同一个文件有修改会造成冲突，此时可能需要手动修改文件
```

