# Git知识总结

廖雪峰老师的git教程：https://www.liaoxuefeng.com/wiki/896043488029600

常用命令：

```js
git add . // 实际上就是把文件修改添加到暂存区；
git commit -m //'实际上就是把暂存区的所有内容提交到当前分支。
git status // 查看修改状态
git diff （文件名） // 查看文件名修改的不同地方位置
git log  // 查看日志版本改变
git reset --hard HEAD^ // 回退上个版本号 HEAD是当版本 HEAD^是上一个版本
git reset --hard 0754c9 // 回退上个版本号后 想要回到之前的版本 可以使用这个命令， 但前提是知道git log 后那个git版本的id好0754c9就是id号
git reflog // 用来记录你的每一次命令
git checkout -- 文件名 // 其实是用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”。
	
git checkout -b dev || git switch -c dev // 创建分支dev
git branch // 命令查看当前分支
git checkkout 分支名 || git switch 分支名// 切换分支
git branch -d dev // 删除分支

git merge 分支名 // 表示需要合并的分支名 
git merge --no--ff -m '内容' 分支名 // 合并分支和上面个一样 区别是--no--ff表示禁用Fast forward模式
 //加上--no-ff参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而fast forward合并就看不出来曾经做过合并。
```

#### 提交

```js
//第一步 提交所有文件
git add .
// 第二部 提交commit 将文件存入缓存区
git commit -m "内容"
```

#### 撤销修改

```js
场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD <file>，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。
```

#### 删除文件

```js
git rm // 在git版本库中删除文件
git commit -m 'rm file' // 并且提交
git checkout -- test.txt // 如果错删了可以使用checkout
```



#### 关联远程仓库并推到远程仓库中

```js
$ git remote add origin github地址 // origin是远程仓库的默认名
$ git push -u origin master // 推到远程仓库中
//由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

git push origin master // 后面提交推到远程仓库就可以使用这个命令了
```



切换分支

```js
git checkout dev // 创建分支
// 新建文件并且修改文件之后要使用git add 文件名以及git commit -m '内容' 后切换分支才会消失
```



解决冲突

```js
//当两个分支同时修改了两个地方，在都提交了时候会发生冲突（比如dev分支和master分支）
//当master分支需要合并了时候会报错
//那么就要删除或者选择是否合并或者删除之类的再在master分支上重新提交

// dev分支上
git add 1.txt
git commit -m 'dev修改'

//切换
git switch master

// master分支
git add 1.txt
git commit -m 'master修改'
git merge dev // 合并

//发生冲突了 选择是否合并或者删除代码 重新提交
git add 1.txt
git commit -m '解决冲突'
```