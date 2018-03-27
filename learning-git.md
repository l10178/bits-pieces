 # Git & GitHub
 github 的分支操作
 首先需要当前目录设置为仓库目录

 一、创建本地分支

 1、查看有哪些分支：git branch

 2、创建一个分支：git branch name  ，其中name是分支名

 3、切换到分支：git checkout name

 说明：可以将2和3合起来操作，创建+切换分支：git checkout -b name

 下面就可以在当前分支上进行相关的文件操作了。

 注意，如果用 git checkout master切换到主分支，在当name分支下进行的文件变更的内容无法看到。当切回name分支后，又可以看到了。



 二、提交分支到github服务器

 git push origin name

 说明：分支提交到服务器上后，如果在本地对分支进行变更后，同样可以执行该操作，将变更信息更新到github的分支上。



 三、将分支的更新内容合并到master分支

 切换到master分支， git checkout master

 合并name分支到当前mater分支：git merge name

 注意：这时合并到master上内容还没有提交到github上，需要push操作。



 四、删除分支

 删除本地分支：git branch -d name

 删除服务器上的分支：git push origin :name   (分支名前的冒号代表删除)



 五、clone分支

 克隆github上的仓库到本地，默认会把仓库的所有内容clone到本地。

 但只会在本地默认创建一个master分支。这时需要用 git branch -r 才能看到所有分支名字。

 这时用 git checkout 分支名 操作就把远程分支取到本地。

 这时再用不带-r的git branch命令就能看到刚才操作的分支名了。
