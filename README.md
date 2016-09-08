## 推荐使用Git bash：最好的Git CLI
######当本地误删除了某文件时：
* 单个文件
     git checkout App1/a.php
* 当前目录所有文件
     git checkout .

###### Undo操作
* 还未执行(git add .)的新增文件操作Undo：git clean -fdx (注意：修改文件和删除文件将不会被undo)
* 执行了(git add .或git commit)的所有操作（增/删/改）的Undo：
     * undo到本地前一次commit的结果(只能一次有效)：git reset --hard
     * undo到本地指定一次commit的结果：git reset --hard HEAD~index（注：index是commit数组的索引,所以是从0开始的，比如HEAD~0指的是最后一次的commit；HEAD~1是倒数第二次的commit；）
     * undo到Server端最后一次commit的结果：git reset --hard @{u}（注：此命令执行时，有可能Server端已经被其他人push过了，但它不会主动undo到最新的版本，所以，理论上讲，该命令会undo到本地最后一次成功push的版本，这个原理其实跟大多数源码管理工具如：svn、VSTS等是一样的，undo操作并不等价于undo + git newest。）
* 所以，一般情况下若为了确保完全undo所有操作，需要连续执行以下2条cmd：git clean -fdx | git reset --hard @{u}.就可以恢复到git服务器版本。

###### check out
* 切换到一个已经存在的分支: git checkout your-branch-name
* 创建一个不存在的分支并切换到该分支: git chekout -b your-branch-name


###### push & pull
* 当执行push推送时，若发现server端的文件与本地不是最新的时，将会push失败，此时执行：git pull拉取最新的server端版本后方可成功。

* 如果git commit了多次，但是都还没有push，这时输入：git push即可全部push，而使用git push origin master则报错。
* 有一次我git的密码修改了，我是执行这个命令：git config --global credential.helper，之后，就会在push时，重新输入密码了
