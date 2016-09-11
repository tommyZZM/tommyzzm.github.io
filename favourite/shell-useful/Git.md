## Git & Github 使用与学习笔记

### 回滚远程仓库

如果你要rollback的commit还没有push到远程仓库的话：
git reset --hard <要rollback到的commit>
如果已经push的话，可以先执行上面的命令，再强制推送你当前的版本到远程仓库：

git push <remote> HEAD --force
执行此命令的时候要小心，确保你的本地仓库是最新的并且没有其他人也在此时做push，否则可能会丢失数据。

### 查看、删除、重命名远程分支和TAG

http://zengrong.net/post/1746.htm

查看远程分支 git branch -a
删除远程分支和tag
 git push origin --delete <branchName>
git push origin --delete tag <tagname>


### 多个sshkey

在 用户文件夹/.ssh/ 创建config文件

如下
````
Host github.com
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_rsa

Host git.***.com
    HostName git.***.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa

Host bitbucket.org
    HostName bitbucket.org
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/bitbucket_rsa
````

即可多个ssh

### 设置一个全局的.gitignore
http://islegend.com/development/setting-global-gitignore-mac-windows/

Assign global .gitignore on Mac
````
git config --global core.excludesfile '~/.gitignore_global'
````
Assign global .gitignore on Windows
````
git config --global core.excludesfile "%USERPROFILE%\.gitignore_global"
````
````
git config --global core.excludesfile "文件路径名"
````

### Git submodule git项目嵌套
开发过程中，经常会有一些通用的部分希望抽取出来做成一个公共库来提供给别的工程来使用，而公共代
码库的版本管理是个麻烦的事情。今天无意中发现了git的git submodule命令，之前的问题迎刃而解了。
添加为当前工程添加submodule，命令如下：

````
git submodule add 仓库地址 路径
````

其中，仓库地址是指子模块仓库地址，路径指将子模块放置在当前工程下的路径。 注意：路径不能以 / 结尾（会造成修改不生效）、不能是现有工程已有的目录（不能順利 Clone）命令执行完成，会在当前工程根路径下生成一个名为“.gitmodules”的文件，其中记录了子模块的信息。添加完成以后，再将子模块所在的文件夹添加到工程中即可。

删除submodule的删除稍微麻烦点：首先，要在“.gitmodules”文件中删除相应配置信息。然后，执行“git rm –cached ”命令将子模块所在的文件从git中删除。

下载的工程带有submodule
当使用git clone下来的工程中带有submodule时，初始的时候，submodule的内容并不会自动下载下来的，此时，只需执行如下命令：
````
git submodule update --init --recursive
````
即可将子模块内容下载下来后工程才不会缺少相应的文件。


###	实用技巧
- [8个小技巧](http://www.oschina.net/translate/8-small-git-tips)
- [10 个迅速提升你 Git 水平的提示](http://www.open-open.com/news/view/12f7b53)
- [你不一定知道的几个很有用的 Git 命令](http://www.java123.net/v/885177.html)
- [10个很有用的高级Git命令](http://developer.51cto.com/art/201308/408029.htm)
