# 使用vagrant搭建跨平台开发环境小记 (windows10/osx) 

### 目的

- 在不同的系统(windows和osX)上**统一开发环境**
- 开发环境傻瓜式移植和搭建,避免重复工作
- 隔离**开发环境**和**系统运行环境**,保持系统整洁
- 开发环境和生产环境保持一致

### 准备工作
首先,参考以下文章,了解和安装vagrant

- [使用 Vagrant 打造跨平台开发环境](https://segmentfault.com/a/1190000000264347)

### 镜像(Ubuntu)

因为`Ubuntu precise`的box镜像,Ubuntu版本是12.0,已经停止维护和更新,所以不建议使用

建议选Ubuntu `14.0`以上的版本
可以在这个github仓库中找到相应的box,[vagrant-box-ubuntu](https://github.com/kraksoft/vagrant-box-ubuntu/)

>如果你要其他系统的镜像，可以来这里下载：[http://www.vagrantbox.es/](http://www.vagrantbox.es/)

下载完box文件后放到某一个目录下,然后添加镜像到vagrant

```
vagrant box add <别名> <.box镜像路径>
```

### 开发环境

常用的包,就直接`apt-get install`安装就好,要最好先使用`apt-get update`进行源更新
以下几个包需要特别说明一下.

#### tmux
```
apt-get install tmux
```
##### 备份tmux会话

建议使用自动备份复原的 `Tmux Continuum` 插件

安装方法：
```
% cd ~/.tmux
% git clone https://github.com/tmux-plugins/tmux-continuum.git
```

接着，将以下内容添加到 `~/.tmux.conf`：

```
run-shell ~/.tmux/tmux-continuum/continuum.tmux 
set -g @continuum-save-interval '20' #设置备份频率(分钟/次)
```

刷新tmux配置
```
# tmux source-file ~/.tmux.conf
# 如果tmux不能识别 `~` 别名,则需要填写绝对路径

tmux source-file /home/username/.tmux.conf
```

`Tmux Continuum`需要1.9版本以上的tmux,使用`tmux -V`查看版本,

`Ubuntu 14.0`以上的版本,apt-get安装时会默认安装1.9以上的版本,低于14.0则可能需要自己手动编译升级

#### vsftpd

vagrant可以使用`synced_folder`进行和**宿主系统**进行目录同步
```
config.vm.synced_folder "/e/local/share", "/home/vagrant/share"
```
但是这里是有一个坑的,如果同步的目录使用了`symbolic link`,
那么软链接在`windows`环境下将不能正常工作,会报出`Protocol error`

所以这里我选用了在虚拟机下搭建一个ftp服务器进行文件同步,以保持更好的兼容性

这里使用的是`vsftpd`
```
sudo apt-get install vsftpd 
```

检测是否正常开启,是否能正常登陆
```
ftp 127.0.0.1
```

关键配置`/etc/vsftpd.conf`
```
listen=YES 
local_enable=YES      #本地用户访问
write_enable=YES  
userlist_enable=YES   #用户列表
userlist_deny=YES     #列出用户不能访问
chroot_local_user=YES #用户是否只能访问默认目录
chroot_list_enable=YES
allow_writeable_chroot=YES
```
参考连接:[Ubuntu下ftp服务器的安装配置](http://blog.csdn.net/delphityro/article/details/22791569)

注意需要添加两个文件(可以为空文件)
- `/etc/vsftpd.chroot_list` #列出名字的用户可以切换到其他目录
- `/etc/vsftpd.user_list` #列出的用户不能访问

通常来说为了保证开发环境的安全让用户访问默认目录就好啦

然后添加一个用户`adduser --shell /bin/false --home /home/<user_name> <user_name>`

ftp搭建完成后,就可以使用虚拟机的ftp地址进行文件同步.

例如在webstorm中可以很方便的使用`Deployment`功能获得开发环境的文件进行编辑.

#### nvm

- [nvm](https://github.com/creationix/nvm)

按照官方仓库的指引走
```
git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && git checkout `git describe --abbrev=0 --tags`
```

```
. ~/.nvm/nvm.sh
```

然后需要这一步,才能找到nvm命令
```
source ~/.nvm/nvm.sh
```

- [Node Version Manager install - nvm command not found](http://stackoverflow.com/questions/16904658/node-version-manager-install-nvm-command-not-found)

### DONE!

然后开发环境就基本部署完成了.

在宿主命令行中
```
vagrant global-status # 查看当前运行实例的hash ID
```

打包
```
cd /e/myboxes # 存储目录
vagrant package <hash> 
```

然后会在当前目录创建一个命名为package.box的镜像文件,
只要在其他系统上使用就可以立刻运行同样环境.

### 常用vagrant命令列表

...
