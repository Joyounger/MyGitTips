
man git-config

###### 客户端基本配置
commit.template 使用设置的文件作为默认的commit-msg

core.pager git在分页输出时所选用的分页程序


core.excludesfile 




help.autocorrect git会自动执行推测出的命令

color.ui 默认auto会对输出到终端的内容进行着色，如果输出被重定向到管道或文件，其中的彩色控制码会被忽略。设为always可以忽略终端与管道之间的差异。不过极少需要这样，绝大多数情况下，如果需要重定向输出中的色彩码生效，可以给git命令传入--color，强制使用色彩码。

###### 格式化与空白字符
1 core.autocrlf
win下可以设置为true，这样在检出代码时就可以将LF转换为CRLF。
Linux/mac下设为input，在提交时将CRLF转换为LF。
如果项目只在win下开发，可以设为false。
2 core.whilespace


###### 服务器配置
receive.fsckObjects
git能够确认在推送过程中接收到的每一个对象的有效性以及是否匹配其sha-1校验和，但这不是默认行为，因为这样成本太高，尤其在处理大型仓库或推送大文件时。
设为true可强制开机检测。

recreceive.denyNonFastForwards
如果设为false,变基时如果确定自己的操作没问题,可以在push时用-f强制更新远程分支.
如果要拒绝这种强制推送,可以设为true.另一种实现方法是通过服务器端的接收钩子,这种方法能完成更复杂的操作,如禁止对某些用户做非快进式推送.
receive.denyDeletes
有一种方法可以绕开denyNonFastForwards策略:先删除某个分支,然后将其连同新的引用一起推回.为避免这种情况,可以将receive.denyDeletes设为true,这样用户就无法删除分支和标签了,要想删除远程分支,必须从服务器上手动删除引用文件.


###### Git属性
也可以为上面讲到的部分设置指定路径,这样Git就可以将这些设置应用于目录或一组文件.
针对路径的设置被称为Git属性,可以在项目的根目录下的.gitattributes中设置,如果不希望将属性文件连同项目一起提交,可以在.git/info/attributes文件中设置
让git识别二进制文件:
在.gitattributes中加入:*.bin binary
git会将所有的bin后缀的文件视为二进制文件,之后在项目中执行git show或git diff时,也不会再计算或输出文件的差异变化

比较二进制文件:
在.gitattributes中加入:*.docx diff=word
然后自定义word过滤器:配置git,让它通过doc2txt将word转为文本文件.
先安装doc2txt,然后编写脚本,
'''
!/bin/bash
doc2txt.pl $1 -
'''
chmod a+x后将脚本放到PATH下,最后配置git使用这个脚本:
git config diff.word textconv doc2txt
现在如果要在两个快照之间比较,且文件以docx结尾,git就会知道对其应用word过滤


比较图片:可以通过一个能提取图像exif信息(大多数图片格式都会保存的元数据)的过滤器来进行对比,可以用exiftool:
*.png diff=exif
git config diff.exif.textconv exiftool


###### 导出仓库
1 export-ignore
对于那些不想包含在项目档案文件中,但是又希望能够剑入到项目中的子目录或文件,可以用export-ignore属性指定,比如不想导出项目的test子目录
test/ export-ignore

2 export-subst
当导出文件用于部署时,可以将git log的格式化和关键字扩展功能应用于使用export-subst属性所标出的部分文件
例如,想将一个名为LAST_COMMIT的文件放入到项目中,并在执行git archive时自动向其注入最新提交的相关数据,可以在.gitattributes中设置:
LAST_COMMIT export-subst

$ echo 'Last commit date: $Fornat:%cd by %aN$' > LAST_COMMIT
$ git add LAST_COMMIT .gitattributes
$ git commit -am 'adding LAST_COMMIT file for archives'




##### Git钩子
克隆项目时不会复制钩子脚本
###### 安装钩子
这些钩子都保存在Git目录下的hooks子目录中,绝大多数项目中,就是在.git/hooks下.
要启用一个钩子脚本,需要将正确命名的可执行文件放入.git目录下的hooks子目录内,这样就可以调用该钩子脚本.

###### 服务端钩子
这些脚本会在向服务器推送之前和之后运行.在推送之前运行的钩子可以随时以非0值退出,拒绝推送操作,并在客户端给出错误提示.
pre-receive  
在处理客户端推送时,首先运行的脚本是pre-receive.可以使用这个钩子来确保所有更新的引用都是快进式的,或是对推送所修改的引用和文件实施访问控制.
update  
update脚本和pre-receive非常类似,不同之处在于前者会为每一个要更新的分支都运行一次.如果试图推送多个分支,pre-receive只会运行一次,而update会为每个推送分支都运行一次.


##### Git强制策略示例
###### 服务器端钩子
所有服务器端的工作都由hooks下的update脚本文件来完成,对于每个推送的分支,update钩子都会运行一次,它接收以下3个参数:
被推送的引用的名称
该分支原先的内容
被推送的新内容
如果允许所有人通过公钥身份验证关联到单一git账号,就需要设置shell脚本根据公钥确定连接的用户身份,并根据情况设置环境变量:
```
#!/usr/bin/env ruby

$refname = ARGV[0]
$oldrev = ARGV[1]
$newrev = ARGV[2]
$user = ENV['USER']

puts "Enforcing Policies..."
puts "(#{$refname}) (#)"
```

强制特定的提交信息格式:
将变量$oldrev,$newrev的值传给git rev-list，就可以得到一个包含所有提交sha1值的列表。
从提交中获取提交消息以供测试：git cat-file commit [commit-id],
```
git cat-file commit [commit-id] | sed '1,/^$/d' #找到第一个空行，提取其后的所有内容
```
可以用这个技巧提取所有待推送提交中的提交消息，如果发现有不匹配的地方退出即可，
```ruby
$regex = /\[ref: (\d+)\]/


#强制的自定义提交信息格式
def check_message_format
 missed_revs = `git rev-list #{Soldrev}..#{$newrev}`.split("\n")
 missed_revs.each do |rev|
  message = `git cat-file commi #{rev} | sed '1,/^$/d'`
  if !$regex.match(message)
    puts "[POLICY] Your message is not formatted correctly"
    exit 1
  end
 end
end

check_message_format
```

















