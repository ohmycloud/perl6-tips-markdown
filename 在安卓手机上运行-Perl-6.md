## 准备工作

- 一只 Root 过的安卓智能手机(例如 Smartisan T1)
- 下载并安装 busybox.apk 、 Linux Deploy.apk 和 JuiceSSH.apk

## 安装 Linux

下面以在我的锤子手机上安装 Debian 为例, 说明如何在手机上运行 Linux:

### 设置 Linux Deploy

打开手机上的 Linux Deploy.apk：


![Linux Deploy](http://upload-images.jianshu.io/upload_images/326727-e26d3ec9f0f75281.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在下面工具栏「启动」->「停止」-> 「下载」 tab 中找到那个类似下载图标的按钮点击, 进入属性设置, 其中属性设置如下所示:


![发行版至安装类型](http://upload-images.jianshu.io/upload_images/326727-97f24c469a53d65f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![安装路径至选择组件](http://upload-images.jianshu.io/upload_images/326727-52e6e4259a203066.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Chroot目录至挂载点](http://upload-images.jianshu.io/upload_images/326727-d812fab30694ce20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


其中,

|选项|值|说 明|
|:----|:---|:-----|
|发行版|Debian|根据自己需要的系统选择|
|发行版本|jsssie|选择稳定版|
|架构|armhf|软件会自省判断 CPU 架构类型|
|镜像地址|http://debian.bjtu.edu.cn/debian/|http://ftp.cn.debian.org/debian/|
|镜像大小|不用填写|默认就行|
|选择组件|只保留 SSH 服务器|手机上用什么桌面环境|
|图形界面|取消勾选|手机上不需要 GUI |
|自定义挂载|勾选|在挂载点那里选择 sdcard0|

设置完成后回到最上面的 Intall(安装 GNU/Linux), 就会开始下载镜像文件了:


![开始下载](http://upload-images.jianshu.io/upload_images/326727-d27228af7d710726.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

注意, 如果中间出现 E, 表示有错误发生, 某些文件下载失败, 需要停掉(Stop)重新开始:

![Error](http://upload-images.jianshu.io/upload_images/326727-a153a030532ae345.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

成功安装不会耗费太多时间, 如果网速够好的情况下, 我是连续下载了好几次都失败, 曾一度放弃不搞了, 可能是因为网络不稳定的原因, 这次选择在夜深人静的12点之后进行, 经过若干次(n>=2， n<=4)后最终下载成功:


![Success](http://upload-images.jianshu.io/upload_images/326727-df34331d9bf011cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个时候, 说明手机中的 Debian 系统已经成功启动了。但是我们怎么进去呢？还记得我们之前安装的 JuiceSSH 吗？ 用这个软件进行连接。首先打开这个软件, 点击「连接」一栏, 进行 SSH 配置:


![连接](http://upload-images.jianshu.io/upload_images/326727-32e1d18c27fe86c5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后点击右下角的加号按钮, 新建一个连接：

![点击右下角的加号](http://upload-images.jianshu.io/upload_images/326727-36ad51a18c1157c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在弹出的「新建连接」窗口中, 在地址栏中填入 `127.0.0.1`：

![新建连接](http://upload-images.jianshu.io/upload_images/326727-bad556143435fef9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后选中「认证」一栏, 在下拉列表中选择「新建」：


![新建认证](http://upload-images.jianshu.io/upload_images/326727-53b8e009b1d9d7cb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

填入用户名 android, 密码 changeme, 然后点击对号按钮完成设置:


![JuiceSSH设置完成](http://upload-images.jianshu.io/upload_images/326727-0b69fd7a1f81d82a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样设置就完成了, 如要进入 Linux 命令行界面, 就点击 JuiceSSH 中的连接 「127.0.0.1」或者你自己设置的昵称, 进入 Debian 8:


![进入 Debian](http://upload-images.jianshu.io/upload_images/326727-f08529d79e74503d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样还没完, 你会发现输入不了中文, 没有 vim 编辑器, git。。。

## 中文环境的配置

进入终端后用 `ls`查看含有中文名的文件时显示 `?????.txt` 的乱码。然后终端里面也输入不了中文。解决方法的步骤如下：

###  重新配置 locales

在终端中输入:

```
dpkg-reconfigure locales
```

使用空格键选中需要安装的本地化的语言,  `zh_CN.UTF-8` 和 ` en_US.UTF-8`。

###  在终端里设置本地化为 `zh_CN.UTF-8`


![服务器端命令](http://upload-images.jianshu.io/upload_images/326727-05549ceb477f5f00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###  安装中文字体

首先安装 vim 编辑器:

```
sudo apt-get install vim
```

 在 `/etc/default/locale` 文件里写入：

```
LANG=en_US.utf8
LC_CTYPE=en_US.utf8
LC_NUMERIC=en_US.utf8
LC_TIME=en_US.utf8
LC_COLLATE=en_US.utf8
LC_MONETARY=en_US.utf8
LC_MESSAGES=en_US.utf8
LC_PAPER=en_US.utf8
LC_NAME=en_US.utf8
LC_ADDRESS=en_US.utf8
LC_TELEPHONE=en_US.utf8
LC_MEASUREMENT=en_US.utf8
LC_IDENTIFICATION=en_US.utf8
LC_ALL=
```

 在 `/etc/environment` 里面写入:

```
LANGUAGE="en_US:en"
LANG=en_US.utf8
LC_CTYPE="zh_CN.utf8"   
LC_NUMERIC="en_US.utf8"
LC_TIME="en_US.utf8"
LC_COLLATE="en_US.utf8"
LC_MONETARY="en_US.utf8"
LC_MESSAGES="en_US.utf8"
LC_PAPER="en_US.utf8"
LC_NAME="en_US.utf8"
LC_ADDRESS="en_US.utf8"
LC_TELEPHONE="en_US.utf8"
LC_MEASUREMENT="en_US.utf8"
LC_IDENTIFICATION="en_US.utf8"
```

然后在 deploy linux 这个软件中重启 Debian 系统, 就是先 Stop，再 Start，重新使用 JuiceSSH 进入 Debian 命令行界面, 再次输入中文就可以了：


![哈哈哈](http://upload-images.jianshu.io/upload_images/326727-829e830b303f05f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 安装 Rakudo Perl 6

在手机上输入一系列命令不太方便, 所以在 Windows/Linux/Mac 下使用 ssh 工具可以连接到手机上, 直接在电脑上操作手机中的终端, 下面以 Windows 为例, 使用 ssh 工具(注意你必须已经安装了它)连接 Debian, 打开 Windows 上的命令行提示符, 输入:

```
ssh android@192.168.1.101
```

或者使用一个批处理文件：

```
@echo off
echo -^> Start login
cmd /k ssh android@192.168.1.101
```
然后会提示你输入密码 changeme, 之后就连接成功了：


![succ.png](http://upload-images.jianshu.io/upload_images/326727-bed78be742dbac15.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后安装 Rakudo：

### 准备

```
apt-get install build-essential git
```

### 从源代码安装

```
git clone git://github.com/rakudo/rakudo.git
cd rakudo
perl Configure.pl --gen-moar --gen-nqp --backends=moar
make
make install
```

安装完成后要注意环境变量的设置, 否则会提示 command not found:

```
echo "export PATH=\$PATH:$HOME/rakudo/install/bin" >> ~/.bashrc
source ~/.bashrc
```

然后退出终端, 再次打开终端输入 `perl6 --version` 就可以使用 perl6了。


![perl6](http://upload-images.jianshu.io/upload_images/326727-28753a43a57d3d2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 安装 panda

```
git clone --recursive git://github.com/tadzik/panda.git
cd panda
perl6 bootstrap.pl
```

然后根据提示:

```
==> Please make sure that /home/android/.perl6/bin is in your PATH
```

把 panda 也加到 PATH 变量中:

```
echo "export PATH=\$PATH:$HOME/.perl6/bin" >> ~/.bashrc
source ~/.bashrc
```

或者编辑 `vim  ~/.bash_profile` 文件:

```
PATH=$PATH:/home/android/rakudo/install/bin:/home/android/.perl6/bin/
export PATH
```

安装 Linenoise 模块

```
panda install Linenoise
```


## 总结

比较容易失败的地方的是系统镜像的下载和中文环境的配置。注意这两点, 就抓住重点确保安装顺利进行了。安装完你可以干什么？ 我先运行个 Python, 微信机器人，Mojo-Webqq, Mojo-Weixin, 写写  jekyll 博客，使用 Perl 6, 都能如你所愿, 因为它既使用 Wifi 也可以使用 3g/4g 蜂窝网络, 随时随地写东西。Once Build, Write Anywhere。 

最后, Just For fun!
