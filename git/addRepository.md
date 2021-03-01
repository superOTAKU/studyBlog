# Git添加仓库

经常有这样的场景：本地写了一个项目，想上传到github上，记录一下具体流程。

1.初始化本地仓库
```bash
PS C:\test> git init
Initialized empty Git repository in C:/test/.git/
```

2.设置远程仓库地址
```bash
PS C:\test> git remote add origin git@github.com:superOTAKU/test.git
```

3.git pull 拉取远程分支
```bash
PS C:\test> git pull origin main
remote: Enumerating objects: 3, done.
remote: Counting objects: 100% (3/3), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (3/3), 580 bytes | 36.00 KiB/s, done.
From github.com:superOTAKU/test
 * branch            main       -> FETCH_HEAD
 * [new branch]      main       -> origin/main
```
此时本地分支情况：
```bash
PS C:\test> git branch -a
* master
  remotes/origin/main
```

4.修改本地分支名称，关联远程分支并推送到github仓库
```bash
PS C:\test> git branch -m master main
PS C:\test> git push --set-upstream origin main
Everything up-to-date
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

成功！
