在提交之前执行git diff --checked，能够鉴别并列出可能存在的空白字符错误。
   --check
           Warn if changes introduce conflict markers or whitespace errors. What are considered whitespace
           errors is controlled by core.whitespace configuration. By default, trailing whitespaces
           (including lines that solely consist of whitespaces) and a space character that is immediately
           followed by a tab character inside the initial indent of the line are considered whitespace
           errors. Exits with non-zero status if problems are found. Not compatible with --exit-code.


git log branchA..branchB 这种语法是一个日志过滤器，它要求git显示出一个提交列表，该列表中的提交出现在后一个分支中，但不存在于前一个分支中。

git merge --squash
 --squash, --no-squash
           Produce the working tree and index state as if a real merge happened (except for the merge
           information), but do not actually make a commit, move the HEAD, or record $GIT_DIR/MERGE_HEAD
           (to cause the next git commit command to create a merge commit). This allows you to create a
           single commit on top of the current branch whose effect is the same as merging another branch
           (or more in case of an octopus).

           With --no-squash perform the merge and commit the result. This option can be used to override
           --squash.

git merge --commit
       --commit, --no-commit
           Perform the merge and commit the result. This option can be used to override --no-commit.

           With --no-commit perform the merge but pretend the merge failed and do not autocommit, to give
           the user a chance to inspect and further tweak the merge result before committing.


git format-patch -M
       -M[<n>], --find-renames[=<n>]
           Detect renames. If n is specified, it is a threshold on the similarity index (i.e. amount of addition/deletions compared to the file’s size). For example, -M90% means Git should consider a delete/add pair to be a rename if more than 90% of the file hasn’t changed. Without a % sign, the number is to be read as a fraction, with a decimal point before it. I.e., -M5 becomes 0.5, and is thus the same as -M50%. Similarly, -M05 is the same as -M5%. To limit detection to exact renames, use -M100%. The default similarity index is 50%.



###### 应用补丁
* git apply采用的是一种要么全有要么全无的模式,也就是补丁全部应用或是一个都不应用， 可以在实际应用补丁之前用git apply --check检查补丁能否顺利应用。如果检查失败，命令会以非0状态退出。git apply适用于diff和git format-patch产生的补丁，对于diff产生的只能用git apply处理。
* 应用由format-patch产生的补丁推荐用git am。如果希望冲突时智能解决冲突，可以用-3选项，这使得git尝试进行3方合并。
       -3, --3way, --no-3way
           When the patch does not apply cleanly, fall back on 3-way merge if the patch records the
           identity of blobs it is supposed to apply to and we have those blobs available locally.
           --no-3way can be used to override am.threeWay configuration variable. For more information, see
           am.threeWay in git-config(1).

* 3点语法(triple-dot synatx):在git diff命令中，可以把3个点号放在另一个分支后，在所处分支的最新提交和两个分支间的共同祖先之间执行git diff master...current
该命令仅会显示出当前主题分支current与master分支的共同祖先之后该主题分支current所引入的内容。

* 将引入的工作从一个分支移动到其他分支的另一个方法是拣选(cherry-pick),cherry-pick类似于对单个提交的变基。

* rerere: reuse recorded resolution(重用已记录的冲突解决方案)的缩写，这是一种简化手动解决冲突的方法。启用rerere之后，git会保留成功合并前后的一组镜像，如果git注意到出现的冲突和之前已经解决过的冲突一模一样，它就会使用之前的解决方案。该特性涉及配置和命令，配置设置是git config rerere.enabled true.现在执行的合并操作解决了合并冲突，解决方案就会记录在缓存中，以备后用。

###### 为发布版打标签
如果打算签署标签，应该使用git tag -s v1.5.
如何分发签署标签的pgp公钥：将公钥作为blob对象包含在仓库中，然后添加一个直接指向其内容的标签。先用gpg --list-keys找出所需要的密钥，然后通过导出秘钥并利用管道将其传给git hash-object命令来实现的，该命令会向git写入一个内含导出内容的新blob对象，并返回这个blob对象的sha-1值：
gpg -a --export xxxxx | git hash-object -w --stdin
现在可以用过执行由hash-object命令给出的新sha-1值来创建一个直接指向该秘钥的标签：
git tag -a xxx sha1xxx
需要验证标签的人可以通过从git数据库中直接取出blob对象并将其导入gpg的方法来导入gpg秘钥：
git show xxx | gpg --import

###### 生成构建号
如果想要为提交起一个具有可读性的名称，可以针对该提交执行git describe。git会给出这样一个名称：由最近的标签名，该标签之上的提交，描述的提交的部分sha1值组成。如果描述的提交已经直接打了标签，git describe只会给出标签名。git describe偏好含有附注的标签(用-a或-s创建的标签)

###### 准备发布
用git archive创建一个包含代码最新快照的归档文件
git archive master --prefix='project/' | gzip > `git describe master`.tar.gz
git archive master --prefix='project/' --format=zip > `git describe master`.zip
如果有人解开这个tar包，就可以在project目录下得到项目最新快照

###### 简报
可以用git shortlog快速得到上次发布或发送电子邮件之后新增内容的各类变更日志，它能汇总给定范围内的所有提交
























