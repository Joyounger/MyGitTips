* 对于两个或更多分支的合并提交，存在多个父提交
* git的分支只不过是个指向某次提交的轻量级的可移动指针(movable ponter).

* 创建分支时，git会创建一个可移动的新指针供你使用。git branch branchname这条命令会创建一个指向当前提交的新指针。
* git中的HEAD是一个指向当前所在的本地分支的指针。git通过维护HEAD知道当前处在那个分支
* git log --oneline --decorate可以查看各个分支当前指向的对象
Print out the ref names of any commits that are shown. 
* git中的分支shi实际上就是一个简单的文件，其中只包含了该分支所指向提交的长度为40个字符的SHA-1校验和。正因为如此git分支创建和删除的成本很低。

* git merge A，将A分支的和当前分支共同祖先后的所有提交合并到当前分支
执行后的提示如果出现“fast-forward”,说明当前分支指向的提交是要合入的hotfix分支的直接上游，因而git会将当前分支指针向前移动。当试图合并两个不同的提交，而顺着其中一个提交的历史可以直接到达另一个提交时，git就会简化合并操作，直接把分支指针向前移动，因为这种单线历史不存在有分歧的工作。
如果提示“merge made by the recursive strategy”,说明git会基于三方合并的结果创建新的快照，然后再创建一个提交指向新建的快照，这个提交叫做“合并提交”。合并提交的特殊性在于它拥有不止一个父提交。git会自己判断最优的共同祖先，并将其作为合并基础。

* git branch输出中，分支名前有*的就是当前分支(既HEAD指向的提交)
* git branch --merged/--no-merged 分别是筛选已并入当前分支的所有分支和筛选尚未并入的所有分支。


###### 远程分支
* 远程分支是指向远程仓库的分支的指针，这些指针存在于本地且无法被移动，当你与服务器进行任何网络通信时，他们会自动更新。远程分支有点像书签，它们会提示你上一次连接服务器时远程仓库中每个分支的位置。
* 远程分支的表示形式是(remote)/(branch) 
* master和origin在git中都没有特殊含义，master是git init时创建的初始分支的默认名称，origin也一样，是git clone时远程仓库默认名称。git clone -o可以自定义远程仓库名
 --origin <name>, -o <name> Instead of using the remote name origin to keep track of the upstream repository, use <name>.

* git fetch会更新本地数据库，然后把本地的origin/master指针移动到最新位置。

* 如果使用https的远程服务器地址进行数据推送，那么git服务器会要求提供用户名密码进行验证。可以设置一个缓存凭据credential cache,最简单的设置方法是把凭据信息暂时保存在内存中几分钟，这只需要zhi执行git config --global crendital.helper cache
* 基于远程分支创建的b本地分支会自动成为跟踪分支(tracking branch),或者有时候也叫作上游分支(upstream branch)
* 跟踪分支是与远程分支直接关联的本地分支。在跟踪分支上执行git push/pull时git会知道要推送拉取哪个远程服务器上那个分支。
* git checkout -t remote/branch 本地创建分支并跟踪远程同名分支。
* 更进一步，用git checkout branch执行切换分支操作时，如果本地还没创建branch，且某个远程分支也叫branch，则git会自动创建跟踪分支。
* 想让git创建的本地已存在的分支设置跟踪分支，或者要更改本地分支对应的远程分支,可以使用git branch的-u或--set-upstream-to设置远程分支
* 如果已经设置好上游分支，可以通过@{upstream}或@{u}的简略写法来使用它。
* git branch -vv可以查看已经设置了哪些跟踪分支，输出中的ahead n表示本地分支上有n个提交还没推送服务端。behind n就是服务器上有一次提交没合并到本地

* git pull在大多数情况下基本等同与git fetch之后紧接着执行了git merge。如果当前分支有跟踪分支，执行git pull时就会拉取并尝试自动合并
* 一般来说，显示的直接用fetch和merge比使用git pull要更好，git pull的机制常常会使人迷惑

* 删除远程分支，git push remote --delete branch。其实只是删除远程服务器上的分支指针，git会保留数据，直到下一次触发垃圾回收


##### 变基(rebase)
变基用git rebase完成，会把某个分支上所有提交的更改在另一个分支上重新一遍。
rebase的工作原理是，首先找到两个要整合的分支(当前所在分支和要整合到的分支)的共同祖先，然后取得当前所在分支的每次提交引入的更改(diff),并把这些更改保存为临时文件，
然后取得当前所在分支的每次提交引入的更改(diff),并把这些更改保存为临时文件，这之后将当前分支重置为要整合到的分支，最后在该分支上依次引入之前保存的每个更改。

使用rebase可以获得更简洁的提交历史，rebase后得到的分支的提交历史看起来是一条线，好像所有的工作都是顺序进行的，即使一开始的情况实际是存在两条平行的开发历史线。
在需要确保你提交的更改能够干净地应用在远程分支上时，经常会用到变基。例如，你想要为某个项目贡献更改，但该项目并不受你控制和维护。在这种情况下，你会在本地进行开发，然后在准备好把补丁提交到项目主干时，就要把工作变基到oeigin/master上，这么做可以让维护者不用去做任何整合工作，而只进行简单的快进合并。

注意不管是变基操作后最新的提交，还是合并操作后最终的合并提交，这两个提交的快照内容是完全一样的，这两种操作的结果区别只是得到的提交历史不一样。



git rebase --onto master server client 将dan当前分支切换到client分支，并找出client分支和server分支的共同祖先提交，然后把自从共同祖先提交以来client分支上独有的工作在master分支上重现。
git rebase basebranch topicbranch 直接对该分支执行变基操作，不需要先切换到该分支。该命令会读取主分支上的更改，并在基础分支上重现

不要对已经存在于本地仓库之外的提交执行变基操作。













