# 下载 git
- https://git-scm.com/downloads

## 常用命令
```sh
git config --global user.name ByDylan-YH
git config --global user.email 921644606@qq.com
git config --global user.password By9216446o6

git config --list

# 初始化版本库
git init
# 设置远程仓库
git remote add dev https://github.com/ByDylan-YH/Notes.git
# 创建本地分支并指向远端分支
git checkout -b dev master
	# 或者正常指定
git branch dev master
# 查看分支
git branch
# 合并分支
git merge dev
# 删除分支
git branch -d dev
# 切换分支
git checkout -b dev
	# 相当于
git branch dev
git checkout dev
	# 新版使用
git switch dev

# 普遍提交流程
	# 添加当前目录下所有文件到缓存区
git add .
	# 如果要丢弃
git checkout -- file
	# 提交缓存区
git commit -m "代码说明"
	# 提交到远程仓库(先下载同步,再上传)
git pull dev master
git push dev master
	# 如果另一用户此时pull报错,则需要
git push --set-upstream origin dev
	# 或者强行覆盖
git push --force by master
# 查看状态
git status
```