
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




































