##GIT分享
svn版本控制系统：

![其他版本控制](http://upload-images.jianshu.io/upload_images/3832193-f6afee5c41a1fdc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 

git 版本控制系统：

![git 版本控制](http://upload-images.jianshu.io/upload_images/3832193-24d0a9a2f98a0d29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 一、git原理 
![MacDown Screenshot](http://upload-images.jianshu.io/upload_images/3832193-9f91b2096b08eeae.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

* 已修改(modified)：表示修改了某个文件，但还没有提交保存  
* 已暂存(staged)：表示把已修改的文件放在下次提交时要保存的清单中  
* 已提交(committed)：表示该文件已经被安全的保存在本地数据库中了 

``` 
需要提交的文件经过add后先都放到暂存区index(或者叫stage)中， 
然后经过commit指令，一次性提交暂存区的所有修改到head。
一旦提交后，暂存区清空，同时若对工作区没有做任何修改，那么工作区就是干净的。
```

#### git命令地图
![一张图git命令](http://upload-images.jianshu.io/upload_images/1967087-8b2a1d26e47655e0.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 二、新建git项目
以本地创建helloworld为例： 

 * 本地建立git版本库
 
 ```
 	➜ mkdir helloworld
 	➜ cd helloworld 
 	➜ git init
 	
 ```
 
 * 在版本库添加`README.md`文件
  
  ```
 	➜ git add README.md
 	➜ git commit -m "README.md for the project"
 	
 ```
 
 * 为版本库添加名为`origin`的远程版本库
 
 ```
 	➜ git remote -v
 	➜ git remote add origin https://github.com/Summer-Zhang/helloworld.git
 ```
 
 * 执行推送，完成版本库初始化
 
 ```  
	git push -u origin master
```
注：同一代码库如需变更远程版本库可先用`git remote remove origin`删除旧库，再用`git remote add origin https://.xxx.git`指向新库。

### 三、git提交代码
以helloworld改动提交进master为例：

 * 将远程git服务器内容更新进本地git
 
 ```
 	➜ git remote update
 	
 ```
 当前操作效果等同于 `git fetch`
 * 更新完后查看当前master与origin/master差距
 
 ```
 	➜ git lol --all
 	
 ```
 ![lol --all](/Users/summer/Desktop/project/8_git分享/249ADFB7-12F8-4BD4-9F85-FDBAED601CB5.png)
 
 注：`alias.lol=log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit`
 
 *  将origin/master分支代码合并进本地master分支
 
 ```
  	➜ git merge --ff-only origin/master
 ```
 当前操作效果等同于 `git pull`
 *  将本地已commit的master代码推送到远程代码库
 
 ```
  	➜ git push origin master
 ```

### 四、git代码回滚
*  本地代码库回滚


```
	➜ git reset --hard commit-id //回滚到commit-id，在commit-id之后提交的commit都去除
	➜ git reset --hard HEAD~3 //将最近3次的提交回滚
```

*  远程代码库回滚

	原理：先将本地分支退回到某个commit，删除远程分支，再重新push本地分支

```
	➜ git checkout the_branch
	➜ git pull
	➜ git branch the_branch_backup  //备份一下这个分支当前的情况
	➜ git reset --hard the_commit_id //把the_branch本地回滚到the_commit_id
	➜ git push origin :the_branch //删除远程 the_branch
	➜ git push origin the_branch  //用回滚后的本地分支重新建立远程分支
	➜ git branch -D  the_branch_backup  //如果前面都成功了，删除这个备份分支
	➜ 
```

### 五、git分支管理及团队协作
#### 分支类型
*  master 主分支：中央仓库应该有一个、且仅有一个主分支。所有提供到生产环境的正式版本。
*  develop 开发分支：用于日常项目开发，包含了项目最新的功能和代码，所有开发内容都在 develop 上进行。
	Git创建Develop分支并切换为当前分支的命令：
	
	```
		-> git checkout -b develop
		-> git checkout master
		-> git merge --no-ff develop
	```
*  feature(功能), release(预发布), hotfix(修补bug)辅助分支: 辅助开发分支，用于应对一些特定目的的版本开发.
	
	```
	feature操作：
		-> git checkout develop
		-> git checkout -b feature_x 
		# 写代码，提交，写代码，提交。。。
		# feature 开发完成，合并回 develop
		-> git checkout develop
		# 务必加上 --no-ff，以保持分支的合并历史
		-> git merge --no-ff feature_xx
		-> git branch -d feature_xx
		
		#如果想要当前分支能保持与 develop 的更新
		-> git rebase develop
	```
	
	```
	release操作：
		-> git checkout develop
		-> git checkout -b release_x 
		# 修复 bug、检查没问题后合并到 master 分支并删除
		-> git checkout master
		# 务必加上 --no-ff，以保持分支的合并历史
		-> git merge --no-ff release_x
		
		#让 release 分支上 bug 修改作用到 develop 分支
		-> git checkout develop
		-> git merge --no-ff release_x
		# 到此，这个 release 分支完成了它的使命，可以被删除了
		-> git branch -d feature_xx
		
	```
	
	```
	hotfix操作：
		-> git checkout master
		-> git checkout -b hotfix_x 
		# 修完 bug 后
		-> git checkout master
		# 务必加上 --no-ff，以保持分支的合并历史
		-> git merge --no-ff hotfix_x
		
		#让 hotfix 分支上 bug 修改作用到 develop 分支
		-> git checkout develop
		-> git merge --no-ff hotfix_x
		# 到此，这个 release 分支完成了它的使命，可以被删除了
		-> git branch -d feature_xx
		
	```
*  
![](https://sfault-image.b0.upaiyun.com/313/602/3136029559-576d5b0b66eac_articlex)

#### 团队协作
#####团队约定：

1. 正式开发的时候每个人本地只需要有两个分支。一个叫develop，一个是自己的那个分支。例如小明开发，你就在本地切换到自己的xiaoming_gittutorial分支然后进行开发。 
2. 每个人可以直接push自己的分支。但是push develop分支的时候。必须先pull 最新的远程develop分支。然后和本地分支合并，清除conflict之后再push。

#####操作准备：
1. 拉取远程develop分支 `git fetch origin develop:develop`
2. 本地创建小明分支`git checkout -b xiaoming_gittutorial`

#####合并操作：
1. 本地切换到develop分支 `git checkout develop`
2. 更新本地develop内容 `git pull`
3. 例如你是小明，那么在pull到远程的develop最新的内容之后，`git merge xiaoming_gittutorial`
4. 如果出现conflict那么清除conflict之后，`git commit -m `.然后把本地develop `git push origin develop` 到远程的develop.
5. 每完成一个功能就提交一次

### 六、通用别名
~/.gitconfig

```
 [alias]
	 co = checkout #切换分支
	 cm = commit  #提交
	 br = branch  #查看分支
	 st = status  #查看状态
	 ad = add  # 添加新增或者修改的文件到git
	 pl = pull  # 拉分支 更新操作
	 ps = push  # 推分支
	 lol = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
```