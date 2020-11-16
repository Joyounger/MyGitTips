
##### 选择修订版本
###### 单个修订版本

* git log --abbrev-commit
       --abbrev-commit
           Instead of showing the full 40-byte hexadecimal commit object name, show only a partial prefix. Non default number of digits can be specified with "--abbrev=<n>" (which also modifies diff output, if it is displayed).

           This should make "--pretty=oneline" a whole lot more readable for people using 80-column terminals.
在保证不出现重复前提下,输出会采用更简单的commit sha1.默认使用7个字符,但为了保证sha1唯一性,在必要时会变得更长.
linux内核最多只需要11个字符就能保证对象sha1的唯一性.一般项目8-10个字符即可.

* 指定某次提交的最直接方法要求有一个指向其的分支引用.随后就可以在任何需要提交对象或sha1值的git命令中使用分支名称了.
git show brach 显示分支中最后一次提交的对象
git rev-parse brach 查看分支指向哪个特定的sha1, rev-parse是为底层操作设计的,并非针对日常操作.

* git会在工作同时在后台保存一份reflog,这是一份最近几个月的HEAD和分支引用的日志:
git reflog
每当分支顶端由于某些原因被修改时,git都会将对应的信息保存在这些临时历史记录中.也可以使用历史记录来指定先前的提交.加入想查看仓库中HEAD在之前第5次的值,可以使用@{n}代来引用reflog的输出:
git show HEAD@{5}
也可用这种语法查看某个分支从前的位置,例如查看master分支昨天在哪
git show master@{yesterday}
这项技术仅对仍包含在reflog中的数据有效,因此无法使用它查找数月前的提交.
类似于git log输出格式的reflog信息:
git log -g
reflog信息只存在与本地,是仓库的操作日志,因此就算是同一仓库的副本,这份日志的内容也不会相同.

* 在引用尾部加上^,git会将其解释为此次提交的父提交
^后可以跟数字,表示第n个父提交,这种语法仅在合并提交时有效,因为只有合并提交才有多个父提交.首个父提交是当你进行合并时所在的分支,第二个父提交是你所合并的分支.
另一种指定祖先的方式是~,它也指向首个父提交.因此HEAD~和HEAD^等效.但HEAD~2表示首个父提交的首个父提交,即祖父提交.


###### 提交范围
* 双点号,可以让git找出不在同一个分支上的提交.
git log master..experiment 输出所有可以从experiment分支中获得而不能从master分支中获得的提交,可以用来查看experiment分支上还有哪些提交没有被合并到master分支上.
如果想使experiment处于最新状态,需要看看接下来需要合并哪些提交:
git log origin/master..HEAD 该语法的另一种常见用途是查看要推送到远端的内容.
这条命令可以显示出处于当前分支,但不处于origin远端master分支上的所有提交.如果执行git push,且当前分支正在跟踪origin/master,那么git log origin/master..HEAD列出的就是将要被上传到服务器的提交.也可以将..一侧的内容省去,让git假定那部分是HEAD.因此git log origin/master..输出结果一样.


* 多点,git允许在引用前加上字符^或--not来指明不包含在其中的提交,因此下面3中表示等效
git log refA..refB
git log ^refA refB
git log refB --not refA
这种语法可以在查询中指定两个以上的引用,这是在双点号语法中无法实现的.比如查询所有在refA或refB中,但不再refC中:
git log refA refB ^refC
git log refA refB --not refC

* 3点, 对于两个引用,这种语法可以指定仅包含在其中任意一个引用中的所有提交.如果想查看属于master或experiment的非公有引用,可以执行
git log master...experiment

git log --left-right master...experiment  --left-right选项显示出提交属于那一侧的分支


##### 交互式暂存

git add -i/--interactive
       -i, --interactive
           Add modified contents in the working tree interactively to the index. Optional path arguments
           may be supplied to limit operation to a subset of the working tree. See “Interactive mode” for
           details.

git可以只暂存文件的某些部分。如果对文件simplegit.rb进行了两处修改，希望只对qizho其中的一处进行暂存，在交互提示符下输入5或p(代表patch)，git会询问要部分暂存哪些文件，然后对于所选择文件的每一个区块，它都会逐个显示出文件的差异并询问你是否要进行暂存。这时输入？会显示一个操作列表。
如果想暂存每个区块，通常可以输入y或n。
部分暂存后可以退出交互模式，然后执行git commit提交部分暂存。
可以直接用git add -p/--patch来启动相同的脚本
       -p, --patch
           Interactively choose hunks of patch between the index and the work tree and add them to the
           index. This gives the user a chance to review the difference before adding modified contents to
           the index.

           This effectively runs add --interactive, but bypasses the initial command menu and directly
           jumps to the patch subcommand. See “Interactive mode” for details.
还可以通过布丁模式，使用命令reset --patch部分重置文件，使用命令checkout --patch检查部分文件，使用命令stash save --patch储藏部分文件。


###### 贮藏与清理
一个干净的工作目录，以及应用在相同分支上并非成功应用贮藏的必要条件，你可以在一个分支上保存贮藏，然后切换到另一个分支，重新应用这些变更。应用stash时，工作目录中也可以包含已修改和未提交的文件，只要无法干净利落地应用任何操作，git都会给出合并冲突信息。
文件变更可以重新应用，但是之前暂存过的文件却不会被再次暂存。要想这样，必须使用加上--index选项的git stash apply命令，告诉该命令重新应用暂存过的变更。执行过这条命令后就会回到原先的状态。
git stash drop用于删除stash

git stash --keep-index 不存储已经用git add暂存过的内容，适用于如果做出了若干修改，但只想先提交其中的一部分，随后再来处理余下的改动的场景。
If the --keep-index option is used, all changes already added to the index are left intact.


git stash -u/--include-untracked stash所有创建过的未跟踪文件，包括已跟踪文件之外的未跟踪文件。

git stash --patch 只要是修改过的内容，git一律不会stash，相反它会以交互式询问哪些需要stash
           With --patch, you can interactively select hunks from the diff between HEAD and the working tree
           to be stashed. The stash entry is constructed such that its index state is the same as the index
           state of your repository, and its worktree contains only the changes you selected interactively.
           The selected changes are then rolled back from your worktree. See the “Interactive Mode” section
           of git-add(1) to learn how to operate the --patch mode.

           The --patch option implies --keep-index. You can use --no-keep-index to override this.

git stash branch branchname 创建一个新的分支，检出在stash工作成果是的提交，重新应用成果，如果应用成功就丢弃stash
       branch <branchname> [<stash>]
           Creates and checks out a new branch named <branchname> starting from the commit at which the
           <stash> was originally created, applies the changes recorded in <stash> to the new working tree
           and index. If that succeeds, and <stash> is a reference of the form stash@{<revision>}, it then
           drops the <stash>. When no <stash> is given, applies the latest one.

           This is useful if the branch on which you ran git stash push has changed enough that git stash
           apply fails due to conflicts. Since the stash entry is applied on top of the commit that was
           HEAD at the time git stash was run, it restores the originally stashed state with no conflicts.



git clean ：从工作目录中删除所有未跟踪过的文件。
Remove untracked files from the working tree
更安全的选择是执行git stash --all来删除全部内容，同时将其以stash形式保存
-f:强制删除
-d：清空子目录
       -n, --dry-run
           Don’t actually remove anything, just show what would be done.

git clean默认只删除没有被忽略的跟踪文件。任何与.gitignore或其他忽略文件中模式匹配的文件都不会被删除。如果连这些文件也想删除，可以用-x
       -X
           Remove only files ignored by Git. This may be useful to rebuild everything from scratch, but
           keep manually created files.


###### 搜索
git grep 在任何提交树或工作目录中方便查找某个字符串或正则表达式。
默认情况下，grep只查找工作目录下的文件
-n 输出匹配位置行号
--count 输出总计信息
-p 所查找到的匹配属于哪个方法或函数
--and 
       --and, --or, --not, ( ... )
           Specify how multiple patterns are combined using Boolean expressions.  --or is the default
           operator.  --and has higher precedence than --or.  -e has to be used for all patterns.
git grep --break --heading 将输出划分成更易读的格式
除了工作目录，git grep可以搜索任意分支


git log -S <string> 只显示出添加过或删除过该字符串的那些提交
           Look for differences that change the number of occurrences of the specified string (i.e.
           addition/deletion) in a file. Intended for the scripter’s use.

           It is useful when you’re looking for an exact block of code (like a struct), and want to know
           the history of that block since it first came into being: use the feature iteratively to feed
           the interesting block in the preimage back into -S, and keep going until you get the very first
           version of the block.

git log -L 展示代码库中某个函数或代码行的历史
       -L <start>,<end>:<file>, -L :<funcname>:<file>
           Trace the evolution of the line range given by "<start>,<end>" (or the function name regex
           <funcname>) within the <file>. You may not give any pathspec limiters. This is currently limited
           to a walk starting from a single revision, i.e., you may only give zero or one positive revision
           arguments. You can specify this option more than once.

###### 重写历史
git没有历史记录修改工具，可以利用变基工具将一系列的提交变基到它们原来所在的HEAD上，而不用移动到新的位置。
git rebase会从命令行中指定的提交开始，自上而下重演每个提交中引入的变更，它将最早的(而非最近的)提交列在顶端，因为这是第一个需要重演的。提交出现的顺序与通常用git log输出的次序相反。


拆分提交：
f7f3f6d change a bit
310xxxx update README and add blame
a5fxxxx add file
假设想将310xxxx update README and add blame拆分成两次：第一次是update README formatting, 第二次是add blame，可以在rebase -i中将要拆分的提交的指示改为edit，然后脚本会将你带回命令行中，这时可以重置要被拆分的提交，获取到已被重置的变更并从中创建多次提交。
保存并退出rebase，git会分别应用di第一次提交（f7f3f6d）和第2次提交（310xxxx）,现在可以使用git reset HEAD^对那次提交重置，这实际上会撤销该提交并使得修改过的文件变成未暂存状态。现在就可以开始暂存并提交文件，直到拥有多次t提交记录为止，当操作结束后，执行git rebase --continue.

如果需要以某种脚本化的方式重写大量提交（如全面修改电子邮件地址或从所有提交中删除某个文件），那么可以使用另外一个历史重写命令。这个命令就是filter-branch。
除非项目还没公开，也没有人在你打算重写的提交基础上开展过工作，否则不应该使用此命令。
* 从所有提交中删除某个文件 
git filter-branch --tree-filter 'rm -f pwd.txt' HEAD
--tree-filter会在每次检出项目后执行指定的命令，然后重新提交结果。
通常最好是在测试分支上执行，确保结果无误后再硬重置主分支。要想在所有分支上执行filter-branch，可以传入--all

git filter-branch --subdirectory-filter trunk HEAD 让trunk子目录成为新项目每次提交的根目录，git会自动删除所有与此项目无关的提交。

全面修改email地址：
git filter-branch --commit-filter '
      if [ "$GIT_AUTHOR_EMAIL" = "now@email.com" ];
      then
             GIT_AUTHOR_NAME="need";
             GIT_AUTHOR_EMAIL="need@email.com";
             git commit-tree "$@";
      else
             git commit-tree "$@";
      fi' HEAD

###### 重置揭秘
1 HEAD
HEAD就是所创建的下一次commit的父提交。
git ls-tree -r HEAD 显示出HEAD快照的目录列表以及其中每一个文件的SHA-1校验和
2 索引
索引是所预计的下一次提交。 

--hard选项是git reset仅有的一个危险用法，也是git会造成数据损坏的极少数情况之一。git reset的其他用法都很容易撤销，唯独--hard不能


checkout与reset之间的区别：
1 不使用路径
与reset --hard不同，checkout不会影响工作目录。checkout更聪明一点，会在工作目录中进行琐碎合并，这样所有未修改过的文件都会被更新。而reset --hard并不会检查，只是简单的进行全面替换。
另一个不同在于更新HEAD的方式，reset移动的是HEAD指向的分支，而checkout移动的是HEAD，使其指向其他分支
2 使用路径
执行checkout的另一种方法是加上文件路径，与reset一样，这种用法不会移动HEAD，就像git reset [branch] file一样，它会使用提交中的文件来更新索引，但是也会覆盖工作目录中对应的文件。

###### 合并的高级用法
git合并时忽略空白字符
-Xignore-all-space 完全忽略空白字符
-Xignore-space-change 将单个或多个空白字符序列视为等同

如果想重置冲突标记并再次尝试解决冲突，就用git checkout --conflict

git log --oneline --left-right --merge 







###### 使用git调试
git blame默认输出中，第三列的时间是commit time而不是author time
如果第一列commit sha1是以^开头，说明这行是在该文件最初提交中的那些行。这次提交出现在文件第一次被加入到这个项目的时候，从那时起这些行就没有再发生变化。
git blame加入-C选项，git会分析所标注的文件并试着找出其中代码片段的原始出处（如果这些代码片段是从别处复制过来）
       -C[<num>]
           In addition to -M, detect lines moved or copied from other files that were modified in the same commit. This is useful when you
           reorganize your program and move code around across files. When this option is given twice, the command additionally looks for copies
           from other files in the commit that creates the file. When this option is given three times, the command additionally looks for copies
           from other files in any commit.

           <num> is optional but it is the lower bound on the number of alphanumeric characters that Git must detect as moving/copying between
           files for it to associate those lines with the parent commit. And the default value is 40. If there are more than one -C options given,
           the <num> argument of the last -C will take effect.

* 二分查找
git biset会对提交历史记录进行二分查找，帮助尽快确定问题是由哪一次提交引起的。
首先使用git bisect启动排查过程，然后使用git bisect bad告诉系统当前提交有问题，接着必须使用git bisect good [good_commit]告诉git最后一次正常状态是在什么时候，然后git会计算你认为的最后一次提交和当前错误提交之间一共出现过大概多少次提交，然后检出中间那一次提交。这时可以进行测试，如果有问题，说明故障是在该提交之前出现的，就继续执行git bisect bad。如果没有说明问题是在该提交之后出现的，就继续执行git bisect good。
排查结束后，应该执行git bisect reset将HEAD重置到排查开始之前的位置。
实际上如果有一个能在项目正常时返回0，错误时返回1的脚本，完全可以将git bisect自动化：
```
git bisect start HAED v10
git bisect run test-error.sh
```
这样会在每个已检出的提交上运行脚本，直到git找到第一个有问题的提交。

###### 子模块
git submodule add [url] 添加一个新的子模块的remote仓库
默认情况下，




###### 打包
git bundle可以将数据打包到单个文件中，这在很多场景上都能用到。它能将所有能够通过git push命令在网络上推送的东西打包成一个二进制文件。
使用时要列出要打包的引用或特定范围的提交，如果像克隆到其他地方，还得加入一个HEAD引用。
git bundle create repo.bundle HEAD master
恢复:
git clone repo.bundle
如果没有在引用中包含HEAD,还得指定-b master或其他引入的分支。
可以在导入仓库之前先检查其中的内容：git bundle verify xxx.bubdle,这个命令确保该文件是一个合法的git文件，并且有必要的祖先完成正确的重组。
git bundle list-heads xxx.bubdle 查看bundle包可以导入哪些分支


###### 凭据存储
git默认不缓存任何内容，所有连接都会提醒你输入用户名和密码
cache模式会将凭据保存在内存中一段时间，绝不会将密码存储在磁盘上，15分钟后会将其从缓存中清除。
store模式将凭据bao保存在磁盘上的纯文本文件中，且永不过期。这意味着除非修改了Git主机的密码，否则永远都不需要重新输入自己的凭据，这种方法的缺点在于密码以明文形式存储在个人主目录下的纯文本文件中。
通过设置git配置，选择上述方法：
git config --global credential.helper cache





















































































































