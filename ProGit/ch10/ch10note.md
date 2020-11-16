
git fsck
 Verifies the connectivity and validity of the objects in the database



###### 环境变量
git使用curl通过http进行网络操作，GIT_CURL_VERBOSE告诉git显示由curl库生成的所有信息。这基本上相当于在命令行中输入curl -v。
GIT_SSL_NO_VERIFY告诉Git不验证SSL证书，如果使用自签名证书通过HTTPS提供Git仓库服务，或是正在搭建git服务器，还没有安装证书，可以开启这个选项
如果HTTP操作的速率低于GIT_HTTP_LOW_SPEED_LIMIT字节/秒，持续时间超过GIT_HTTP_LOW_SPEED_TIME秒，git将终止操作。这些值会覆盖http.lowSpeedLimit和http.lowSpeedTime配置值。
当git通过HTTP通信时，GIT_HTTP_USER_AGENT设置其所使用的user-agent字符串。


###### 调试
GIT_TRACE 控制一般的追踪，这种追踪不属于任何特定的分类。

GIT_TRACE_PACKET允许在数据包层面上对网路操作进行追踪。

GIT_TRACE_PERFORMANCE控制性能相关数据的日志记录。

GIT_TRACE_SETUP显示了Git了解到的有关仓库以及交互环境的相关信息。



###### gui
gitk是一款图形化历史记录查看工具，可以看做是基于git log和git grep的一个功能强大的GUI shell，调用方式：
$ gitk [git log option]
gitk把大部分选项都传给了底层的git log命令，最有用的选项之一就是--all，它告诉git显示所有引用中的提交（而不仅仅是HEAD）

git-gui主要用于生成提交


###### bash中的git
自定义命令行提示符来显示当前目录所属仓库的相关信息：将git源码中的文件contrib/completion/git-prompt.sh从Git的源代码仓库中复制到主目录下，在.bashrc中加入：
```
. ~/git-promtp.sh
export GIT_PS1_SHOWDIRTYSTATE=1
export PS1='\w$(__git_ps1 " (%s)")\$ '
```
\w表示当前目录，\$输出命令提示符中的$,__git_ps1 " (%s)"使用格式化参数调用由git-prompt.sh提供的函数









