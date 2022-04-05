# git学习
	git是全球最先进的分布式版本控制系统



## 安装git：

	下载并安装git
	设置用户和邮件：  	
		$ git config --global user.name "Your Name"
		$ git config --global user.email "email@example.com"  
	创建版本库repository（可以简单理解为一个目录）



## 初始化

​		在想要初始化为git仓库的文件夹下面启动git bash，然后使用如下命令，将该文件夹初始化为git repository

```
$ git init
```



## 使用，添加文件  

	放在GitRepository目录下
	两步：
```
$ git add file1.txt
$ git commit -m "add 3 files."
```



## 删除文件

​		git rm <file>



## 查看状态，具体修改内容

```
$ git status
$ git diff HEAD -- readme.txt
```



## 查看修改历史，日志（第二行为，将日志按行显示，直观）：

```
$ git log
$ git log --pretty=oneline
```



## 回退版本

​		HEAD表示当前版本，最新版本，在后面加一个^则表示前一个，加^^则表示前两个，多版本的加 ~num (例如下面演示的为回退10个版本)

```
$ git reset --hard HEAD^
$ git reset --hard HEAD~10
```

​		想要再回到刚刚推出的版本则只需记下当时版本的版本号，然后回退(592a即为版本号前4位，git会自动寻找)

```
$ git reset --hard 592a
```

​		若未记录下当时版本commit id，可以使用指令记录功能(该功能会返回使用过的每次命令)：

```
$ git reflog
```



## stage -- 暂存区      ||     master 

​		stage每次git add 先添加到这个地方  

​		然后一次git commit 统一添加到master

​		git commit只提交已经在stage中的文件，即指提交已经add的文件

​		使用以下命令可以查看分别在stage和master中的readme.txt的版本

```
$ git diff HEAD -- readme.txt
```



## 撤销

​		1.没有`git add`时（即为退回到修改之前，即将修改放弃，恢复到上一次commit的状态），用

```
$ git restore file	或
$ git checkout -- file
```

​		2.已经`git add`时（即为退回到add之前，即未add状态，即撤销add操作），

```
先`git reset HEAD `回退到1.，再按1.操作  或
$ git restore --stage file
```

​		3.已经`git commit`时（即为退回到未commit之前，即未commit状态），用`git reset`回退版本

```
$ git reset --hard HEAD^
```

​		推送到远程库，GG?



## 推送远程

​	关联github库 远程库：

  - 先在GitHub上新建一个repository

  - 然后在本地关联此远程库

    ```
    $ git remote add origin git@github.com:x43125/库名.git  (尽量使用这个，不会弹验证)或
    $ git remote add origin https://github.com/x43125/gitLearn.git
    ```

  - 最后将本地更新push到远程库

    ```
    $ git push -u origin master
    ```

    

## 分支

​		创建分支

```
$ git checkout -b dev  或
$ git switch -c dev
（-b表示创建并且换 相当于：
    $ git branch dev
    $ git checkout dev -- git checkout branchName 相当于切换到分支
    $ git switch master）
```

​		作业，提交

​		切换回master分支

​		融合分支

```
$ git merge dev  (合并指定分支到当前分支)
```

​		删除分支

```
$ git branch -d dev
```



## git建立双远程仓库

​		删除原先的origin库，然后重新建立双远程仓库

```
$ git remote add github git@github.com:x43125/项目名.git
$ git remote add gitee git@gitee.com:x43125/项目名.git
```

​		push

```
$ git push github master
$ git push gitee master
```





## 注： 

- 因为创建、合并和删除分支非常快，所以Git鼓励你使用分支完成某个任务，合并后再删掉分支，这和直接在`master`分支上工作效果是一样的，但过程更安全。 

- 当远程库与本地库未建立正确连接时，远程库已有文件，在push本地库之前需要先将远程库的文件pull下来，在push，如pull失败出现fatal:需要强行pull：如下

  ```$ git pull origin master --allow-unrelated-histories ```

- 第一次push时必须加 -u

- git remote -v  查看远程仓库连接





