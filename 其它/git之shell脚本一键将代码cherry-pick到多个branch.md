### 其它
工作中因为产品以及项目本身的特性，我们同一个项目会存在多个版本，不同客户可能使用不同版本的产品，因此发现bug或者有紧急需求，代码需要合并到多个版本，我们用branch分支管理版本，这时候需要一个一个分支去cherry-pick很是麻烦，所以开发了下面一个shell脚本，再也不需要手动去切换版本cherry-pick代码了，实现简单的一键合并代码，**仅用来替代cherry-pick**

### 脚本代码
我们命名脚本文件名称是cherry-pick.sh
```shell
#!/usr/bin/bash

if [ $# -eq 0 ]
then
    echo "cherry-pick hashcode can not empty"
    exit
fi

br=`git branch | grep "*"`
git fetch
branchlist=(20.2.2 20.2.1 20.1.1 19.10.1)   #版本号列表
for branch in ${branchlist[@]}
do
    git checkout branch/${branch}
    git reset --hard HEAD #清理工作区内容
    git rebase origin/branch/${branch}
    if [ $? -ne 0 ]
    then
        echo "git rebase origin/branch/${branch} error"
        exit
    fi
    
    git cherry-pick $*
    if [ $? -ne 0 ]
    then
        echo "git cherry-pick {$*} for branch/${branch} error"
        exit
    fi
    git push origin branch/${branch}
done
#重新切换到当前分支
git checkout ${br/* /}
```
### 使用方式
- ./cherry-pick.sh 哈希值,举例如下
```
./cherry-pick.sh 1b40a0ce89
```
- ./cherry-pick.sh 多个连续的哈希值,举例如下
```
./cherry-pick.sh 1b40a0ce89 4b93149554 51533b9f80 c2cbea077f
```
**注意**
- 若合并过程中存在冲突的版本，则会中断合并，需要手动解决冲突
- 合并成功之后会重新切换到执行命令前所在的分支