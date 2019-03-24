* git diff --staged等同于git diff --cached
* git commit -a Tell the command to automatically stage files that have been modified and deleted, but new files you have not told Git about are not affected.相当于提交前不用执行git add

* git rm 把文件从工作目录移除。如果更改了某个文件，并已经暂存，要想yichu移除必须使用-f选项
× git rm --cached 把文件保留在工作目录，但从暂存区中移除该文件
× git mv用于重命名文件

* git log
-author=<pattern>, --committer=<pattern> 
Limit the commits output to ones with author/committer header lines that match the specified
--grep=<pattern>
Limit the commits output to ones with log message that matches the specified pattern.With more than one --grep=<pattern>, commits whose message matches any of the given patterns are chosen
--all-match
Limit the commits output to ones that match all given --grep, instead of ones that match at least one.
-S<string> 输出那些添加或删除zhidi指定字符串的提交
Look for differences that change the number of occurrences of the specified string
It is useful when you’re looking for an exact block of code (like a struct), and want to know the history of that block since it first came into being: use the feature iteratively to feed the interesting block in the preimage back into -S, and keep going until you get the very first version of the block.

* 撤销已暂存的文件 git reset HEAD <file>...
* 撤销未暂存的修改 git checkout -- <file>...


* 检查远程仓库 git remote show [remote-name]



* git tag -l 搜索特定标签
List tags. With optional <pattern>...


* 创建注释标签 git tag -a,git show tagname可以查看标签数据及对应的提交
* 创建清量标签 这种标签基本上就是把提交的校验和保存到文件中，除此之外不包含其他任何信息。执行git show除了提交信息外，看不到别的信息
* 补加标签，在创建标签命令最后加上提交的commit id即可
* 共享标签：git push默认不会把标签传输到远程服务器上，创建了标签后必须手动将标签推送到远程服务器。这个过程有点像推送分支，命令为
git push [remote] [tagname].如果有很多标签要一次性推送，可以使用git push的--tags选项git push [remote] --tags






