---
layout: post
title: git服务器的建立
excerpt: "git服务器的建立"
modified: 2014-10-12
tags: [编程]
comments: true
image:
  feature: sample-image-4.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
---



# 环境

- 虚拟机Ubuntu12.04
- 客户机：Win7 64位

# 步骤

1，在Ubuntu上`($)`：

    sudo apt-get install operssh-server openssh-client
    sudo apt-get install git-core


2,添加用户和密码`($)`:

    sudo useradd -m git
    sudo passwd git 

上面这两步的目的是为了服务器端管理将来的git工程。我们知道linux系统都有一个root用户，也就是最高级的用户，拥有最高权限，由于root用户比较特殊，权限高，在是用中可能会误操作别的用户的内容或系统的一些文件，所以我们linux系统一般都会有一个日常使用的用户，一般不会登陆root用户，日常用户是为了日常操作使用的，所以我们这个地方又创建了一个用户，用户名是git，专门用来处理git相关事务。这样可以更有条理的工作，要是所有的东西都在日常用户或是root用户下，那么工作目录很乱也可能导致一些操作影响到其他的内容。

3，建立仓库文件夹

    sudo mkdir /home/git/repositories
    sudo chown -R git:git /home/git/repositories
    sudo chmod 755 /home/git/repositories
    git config --global user.name "myname"
    git config --global user.email myname@gmail.com

4,安装gitosis，管理ssh密钥

    sudo apt-get install python-setuptools
    cd /tmp
    git clone https:github.com/res0nat0r/gitosis.git
    cd gitosis
    sudo python setup.py install

# 在win7上

1，下载安装git for windows

2，打开git bash （ip是Ubuntu系统的ip）
    ssh git@192.168.1.1

如果成功，会让你输入密码，就可以登录到服务器了。

3，生成本地计算机的密钥

    ssh-keygen -t rsa

然后回车，系统会做一些提示，全部回车同意就行，最后会生成一个保存密钥的文件，看一下生成保存的位置。在windows上一般位置是保存在c:/Users/(你当前用户名)/.ssh目录下[根据提示，有些默认存储地址不是这个]，有一个id\_rsa(私人密钥，保存好) 和 id\_rsa.pub(公共密钥，要发送给服务器，用来辨别你的身份)。你可以打开看一下这两个文件，里面就是一堆字符。把id_rsa.pub拷贝到服务器上,拷贝到ubuntu系统的/tmp文件夹中。

# 在Ubuntu系统上

    sudo -H -u git gitosis-init < /tmp/id_rsa.pub
    sudo chmod 755 /home/git/repositories/gitosis-admin.git/hooks/post-update

# 在win7上

    git clone git@192.168.1.1:gitosis-admin.git

这会得到一个名为 gitosis-admin 的工作目录，gitosis.conf 文件是用来设置用户、仓库和权限的控制文件。keydir 目录则是保存所有具有访问权限用户公钥的地方― 每人一个。在 keydir 里的文件名（比如上面的 scott.pub）应该跟你的不一样 ― Gitosis 会自动从使用 gitosis-init 脚本导入的公钥尾部的描述中获取该名字。
看一下 gitosis.conf 文件的内容，它应该只包含与刚刚克隆的 gitosis-admin 相关的信息：

现在我们来添加一个新项目。为此我们要建立一个名为 mobile 的新段落，在其中罗列手机开发团队的开发者，以及他们拥有写权限的项目。由于 'scott' 是系统中的唯一用户，我们把他设为唯一用户，并允许他读写名为 iphone_project 的新项目：
    
    [group mobile] 
    members = scott 
    writable = iphone_project
    
    git commit -am 'add iphone_project and mobile group'
    git remote add origin git@gitserver:iphone_project.git
    git push origin master

请注意，这里不用指明完整路径（实际上，如果加上反而没用），只需要一个冒号加项目名字即可 ― Gitosis 会自动帮你映射到实际位置。

要和朋友们在一个项目上协同工作，就得重新添加他们的公钥。不过这次不用在服务器上一个一个手工添加到 ~/.ssh/authorized_keys 文件末端，而只需管理 keydir 目录中的公钥文件。文件的命名将决定在 gitosis.conf 中对用户的标识。现在我们为 John，Josie 和 Jessica 添加公钥：

    cp /tmp/id_rsa.john.pub keydir/john.pub
    cp /tmp/id_rsa.josie.pub keydir/josie.pub
    cp /tmp/id_rsa.jessica.pub keydir/jessica.pub

然后把他们都加进 'mobile' 团队，让他们对 iphone_project 具有读写权限：

    [group mobile] 
    members = scott john josie jessica 
    writable = iphone_project

如果你提交并推送这个修改，四个用户将同时具有该项目的读写权限.Gitosis也具有简单的访问控制功能。







