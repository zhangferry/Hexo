---
title: 基于Hexo搭建自己的博客小屋
date: 2016-12-20 18:50:55
tags: 
    - Hexo
categories: Hexo

---
>作为一名技术人员没有属于自己的博客，就像是喜欢LOL的玩家却没有一款炫酷的皮肤一样，这不叫真爱。虽然现在是微博的时代，讲究方便阅读，易传播，但是对于博客来说，特别是技术博客，专业性永远都是第一位的。我们需要用大大的篇幅去阐述自己对技术的理解并将其分享给其他人，所以无论社交软件如何发展，我们都需要博客。下面就跟着我一块搭建属于自己的博客小屋吧。
<!--more-->
## 搭建环境
已经安装Git的Mac电脑，这个默认都能满足，所以就不详细介绍了。

## 创建github page
首先注册[github](https://github.com)账号，然后在repository选项卡里New一个新的仓库来存储我们的网站

![01.png](http://upload-images.jianshu.io/upload_images/1059465-7ea5a5bdf68d6170.png)

然后命名为username.github.io。

![02.png](http://upload-images.jianshu.io/upload_images/1059465-6fe4e7bfeaeecca3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 安装Hexo
在安装Hexo之前我们需要安装nvm和Node.js。
* Hexo是目前很流行的博客管理框架，基于Node.js
* nvm是Node.js的版本管理工具
* 而Node.js是一个基于 [Chrome V8](https://developers.google.com/v8/) 引擎的 JavaScript 运行环境。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。

不太理解和想要深入了解各软件作用的同学可以自行google，接下来我们开始安装这些东西（确实挺多的）。

1、**通过curl方式安装node版本管理工具nvm**

    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash
其他方式的安装可以自行google。

2、**配置环境变量**
完成之后nvm就被安装在了`~/.nvm`下，接下来配置环境变量
在`~/`目录下看是否有`.zshrc`,`.bash_profile`,或者`.profile`,如果没有就新建一个`.profile`文件。
注意：`.`开头的文件是隐藏文件，在终端查看的时候使用命令`ls -a`,然后打开对应的配置文件在最后一行加上：

    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm
这一步的目的是每次新打开一个bash，nvm都会被自动添加到环境变量中。
3、**验证nvm安装**
在命令行输入`nvm`看到如下信息：

    Node Version Manager

    Note: <version> refers to any version-like string nvm understands. This includes:
    - full or partial version numbers, starting with an optional "v" (0.10, v0.1.2, v1)
    - default (built-in) aliases: node, stable, unstable, iojs, system
    - custom aliases you define with `nvm alias foo`

    Usage:
    nvm help                                  Show this message
    nvm --version                             Print out the latest released version of nvm
    nvm install [-s] <version>                Download and install a <version>, [-s] from source. Uses .nvmrc if available
    --reinstall-packages-from=<version>     When installing, reinstall packages installed in <node|iojs|node version number>
    nvm uninstall <version>                   Uninstall a version
    nvm use [--silent] <version>              Modify PATH to use <version>. Uses .nvmrc if available
    nvm exec [--silent] <version> [<command>] Run <command> on <version>. Uses .nvmrc if available
    nvm run [--silent] <version> [<args>]     Run `node` on <version> with <args> as arguments. Uses .nvmrc if available
    nvm current                               Display currently activated version
    nvm ls                                    List installed versions
    nvm ls <version>                          List versions matching a given description
    (usually `~/.nvm`)
那么恭喜你！nvm安装成功了。这一步在我看来是最容易出错的。

4、**安装node.js**
如果上面的步骤完成了，node.js的安装就简单多了，直接：

    nvm install 4
这个指令是安装nodev4.7.0版本
安装成功后可以使用nvm ls查看当前node版本号

5、**安装Hexo**
安装Hexo也比较简单

    sudo npm install hexo-cli -g

## 配置Hexo站点
完成所需组建的安装，接下来就要建立本地站点，配置站点了。

1、**本地新建博客目录**
目录可以自由选择，我选择在主目录下：

    ~$ mkdir username.github.io
    ~$ cd username.github.io
    ~$ hexo init username.github.io

2、**配置站点**

在站点下有一个`_config.yml`，这里我们可以进行一些对博客的配置

    language: en #语言设置
    theme: next #主题设置，因为下面将使用next主题
    deploy:
    type: git
    repo: https://github.com/username/username.github.io.git

这里的repo就是我们新建仓库的git地址，之后发布的时候就会将内容发布到这个地址下。更多设置可以查看[更多Hexo配置](https://hexo.io/zh-cn/docs/configuration.html)

3、**配置主题**

![03.png](http://upload-images.jianshu.io/upload_images/1059465-14d7e7cf2fecebda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我使用的是目前最受欢迎的一款Hexo主题[Next](http://theme-next.iissnan.com/)使用它的话，我们需要先把它clone到本地

    $ cd username.github.io
    $ git clone https://github.com/iissnan/hexo-theme-next themes/next
在theme文件夹内也有一个`_config.yml`文件，这里是用来配置主题的，[详细设置](http://theme-next.iissnan.com/theme-settings.html)

## 新建、发布博客

经过上面的努力终于可以开心的写博客了，Hexo博客是基于Markdown格式编译的，所以，我们需要了解常用的Markdown语法，不了解Markdown的可以点这里参考[Markdown](https://segmentfault.com/markdown#articleHeader15)，以下命令均在博客站点目录操作

1、**新建**

    hexo new "my blog"
文件生成在`username/source/_posts/my-blog.md`，打开文件，利用markdown语法将内容写到里面。

2、**编译**

    hexo generate //可以简写为hexo g
这一步的作用是将刚才的markdown语法的博客内容编译成html语言。编译之后生成`public`文件夹，里面放的是生成的html文件。之后同步到github上的就是这个文件夹的内容。

3、** 开启本地服务**

    hexo server //可以简写hexo s
这个命令的作用是开启本地服务。之后会有下面两条语句生成

    INFO  Start processing
    INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
我们就可以访问 <http://localhos:4000/>预览博客内容了。

4、**部署**

    hexo deploy //可以简写为 hexo d
部署的作用就是将博客内容发布到网络。执行完成之后我们就可以访问<http://username.github.io>了，当你能够看到自己写的内容呈现在自己眼前的时候有没有很激动呢。哈哈

5、**清楚public内容**

    hexo clean 
这个命令用在当我们更改source内部的资源路径之后，执行此命令可以重新编译生成public文件夹。

---
好了，讲解到此结束，下一篇讲解如何发布博客到指定域名。这个是我的博客<http://zhangferry.tk>，欢迎访问
