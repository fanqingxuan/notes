### 其它
### Git SSH Key 生成步骤
- 设置Git的user name和email
```shell
git config --global user.name "gitadmin"
git config --global user.email "gitadmin@admin.com"
```
- 生成SSH密钥过程
```shell
ssh-keygen -t rsa -C "gitadmin@admin.com"
按3个回车，密码为空。
```
最后得到了两个文件：id_rsa和id_rsa.pub

- 添加密钥到ssh
将id_rsa.pub文件里面的内容复制到自己的git服务端，如github的个人设置里面的ssh相关菜单

### 开始使用
建立本地<-->远端仓库的关系，
如果是新项目(远端不存在该项目)，执行如下
```shell
echo "# demo" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/fanqingxuan/demo.git
git push -u origin master
```
远端已经存在的项目
```shell
git clone git@github.com:fanqingxuan/di.git
```
###设置别名
```shell
git config --global alias.ll "log --graph --all --pretty=format:'%Cred%h %Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative"
git config --global color.status auto
git config --global color.branch auto
git config --global color.diff auto
git config --global color.interactive auto
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.last "log -1 HEAD"
git config --global alias.df diff
git config --global color.ui true
git config --global core.quotepath false   #防止git status中文文件名乱码
git config --global pull.rebase true #设置pull默认rebase
git config --global core.filemode false #忽略文件权限的变更
```
### 常用命令
```shell
git push origin master #将本地的master分支推送到origin主机的master分支,如果后者不存在，则会被新建

#分支支推送顺序的写法是<来源地>:<目的地>，所以git pull是<远程分支>:<本地分支>，而git push是<本地分支>:<远程分支>

#删除远程master分支
git push origin :master
git push origin --delete master

#不使用fast forward合并,日志是一条线，没有分叉
git merge --no-ff -m "merge with no-ff" develop

git add file  #将内容由工作区提交到暂存区
git commit -m "log message"  #将内容由暂存区提交到仓库
git commit -a #由工作区直接提交到仓库
git commit --amend #修改最后一次修改信息,或者是有漏掉的提交，可以将上一次的提交和漏掉的文件重新作为一次提交
git checkout -- file    #用暂存区或仓库内容覆盖工作区，以最新的时间点为参照，这个命令改变的是工作区
git checkout HEAD #用仓库内容直接覆盖工作区
git checkout - #跳到之前的分支
git reset --hard HEAD #工作区和暂存区都回退到和仓库一致

git diff #对比工作区和暂存区
git diff HEAD #对比工作区和仓库
git diff --cached #对比暂存区和仓库

git push origin master:master #本地master仓库提交到远程master仓库

git pull/rebase origin master:fanxiajie #拉取远程master仓库到本地fanxiaojie仓库,并合并到当前分支
git fetch origin fanxiaojie:fanxiaojie #只拉取远程仓库到本地仓库，不合并到当前分支

#把暂存区的修改撤销掉（unstage），重新放回工作区
git reset HEAD
git reset HEAD filename

git add -a #表示添加所有内容， 
git add . #表示添加新文件和编辑过的文件不包括删除的文件; 
git add -u #表示添加编辑或者删除的文件，不包括新添加的文件

#新建标签
git tag v1.0
#给commit id 为25656e2的历史版本打标签
git tag v1.0  25656e2
#查看标签
git tag
#删除本地tag
git tag -d v0.9
#删除远程tag
git push origin :refs/tags/v0.9
#git show tagname查看标签信息
git show v1.0

# 删除 untracked files
git clean -f
# 连 untracked 的目录也一起删掉
git clean -fd
# 连 gitignore 的untrack 文件/目录也一起删掉 （慎用，一般这个是用来删掉编译出来的 .o之类的文件用的）
git clean -xfd
# 在用上述 git clean 前，墙裂建议加上 -n 参数来先看看会删掉哪些文件，防止重要文件被误删
git clean -nxfd
git clean -nf
git clean -nfd

#查看远程仓库地址:
git remote -v
```

### 问题
- git bash命令行下面文件名是中文显示乱码，执行
```shell
git config --global core.quotepath false
```
- git提示"bad index file sha1 signature fatal: index file corrupt"
    需要删除.git/index文件，然后在仓库目录下运行git reset，重新生成index文件
```shell
rm -f .git/index
git reset
```
- git push代码到github上一直提示输入用户名及密码的问题
出现这种情况的原因是我们使用了http的方式clone代码到本地，相应的，也是使用http的方式将代码push到服务器
    git remote -v 查看当前方式
    解决办法很简单，将http方式改为ssh方式，操作步骤如下:
```bash
git remote rm origin
git remote add origin git@github.com:fanqingxuan/blog_demo.git
git push --set-upstream origin master
```
- 进个项目文件夹中，使用git命令时报如下错
```shell
$ git status
fatal: not a git repository (or any of the parent directories): .git
```
实际上，项目文件夹中，存在`.git`文件夹
**原因**:电脑异常关机，电脑运行时间过长，或者非正常退出git，导致git的HEAD文件损坏
**解决方式**:将项目中的.git/HEAD文件内容修改为`ref: refs/heads/develop`,然后重新进入终端