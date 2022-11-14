- 查看所有分支： git branch -a
- 在本地新建一个分支： git branch branch_Name
git checkout -b branch_Name //新创建分支并切换
- 切换到你的新分支: git checkout branch_Name
- 将新分支发布在github上： git push origin branch_Name
- 在本地删除一个分支： git branch -d branch_Name
- 在github远程端删除一个分支： git push origin :branch_Name (分支名前的冒号代表删除)

- 首先切换到main分支上
git  checkout main

- 如果是多人开发的话 需要把远程master上的代码pull下来
git pull origin main
- 如果是自己一个开发就没有必要了，为了保险期间还是pull

- 然后我们把dev分支的代码合并到main上
- git  merge dev

- 删除本地分支 Chapater6
git branch -d  Chapater6