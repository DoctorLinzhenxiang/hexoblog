---
title: Hexo异地同步更新博文
date: 2016-10-22 20:57:23
categories: Hexo
tag: Hexo
description: 
---

Hexo搭建博客目前已经非常流行了，各种教程在网上可以找到一大推，但是，由于不是B/S类型的，对于异地更新博文就比较麻烦了，需要预先搭建相关的环境，另外由于版本的更新还会遇到各种问题，最近就因为重装了一下系统，需要重新搭建环境也就淌过了不少的坑，现在总结下来以备忘。

# 新机器环境搭建

首先，就像刚开始搭建hexo一样，我们需要安装node.js和git的环境，去相关主页上下载最新的安装包傻瓜式安装即可。

1. [node.js环境安装](https://nodejs.org/en/)

2. [git环境安装](https://github.com/git-for-windows/git/releases)

这里需要注意，关于github的配置，需要将新机器的SSH公钥添加到github的[Account settings -> SSH Keys -> Add SSH Key]中去。首先，在本地为git配置用户名和邮箱

```
git config --global user.email "heimingx@gmail.com"
git config --global user.name "Heimingx"
```

接着生成密钥：

```
ssh-keygen -t rsa -C "heimingx@gmail.com"
```

一路回车，通常会在`/.ssh/`目录下生成两个文件id_rsa和id_rsa.pub，将id_rsa.pub文件中的内容复制到github的key文本框中。

最后，我们可以通过

```
ssh -T git@github.com
```

来验证一下。

# Hexo安装和博客的clone

首先，我们在准备存放blog的文件夹内执行clone命令，将事先在github上单独存储的所有blog文件clone到本地。

> 注：这里我们可以在github上另建一个单独的Repository来存储博客的所有原始文件，一方面保存，另一方面方便异地更新。

```
git clone https://github.com/HeimingX/hexoblog
```

然后，我们需要安装hexo，在该文件夹下执行安装命令 

```
npm install hexo-cli -g
```
	
接着，安装hexo的相关插件
	
```
npm install hexo --save
```

可以通过 

```
hexo -v
```

来检查是否安装成功。

此时，通常应该就OK了，可以尝试一系列熟悉的hexo命令来进行创作博文了。

# 踩过的坑

在上面的操作都按部就班的完成之后，我们通常可以执行hexo的相关命令了，但是，不知道什么原因，其他的命令都可以顺利执行，本地也能起Server预览，唯独`hexo deploy`不行，报错：

```
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': Invalid argument
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': Invalid argument
```

在网上查了很多相关的资料，发现也有同学遇到过之类的问题，不过给出的答案比较多，一一试了也木有能够解决。最终，我的解决方案是修改了本地的_config.yml文件的deploy配置。

```
deploy: 
  type: git
  repository: git@github.com:Heimingx/Heimingx.github.io.git
  branch: master
```

之前的repository配置的是 `https://github.com/Heimingx/Heimingx.github.io.git`，具体的原因还有待探究。
这里改完之后，就可以顺畅的操作各种hexo命令了，另外发现还不需要每次都输入用户名和密码了，虽然第一反应是不安全，不过这样也方便了很多，毕竟每次输密码还是挺烦的。

# references
1. [hexo 3.0 为什么无法部署到github上](https://segmentfault.com/q/1010000003823346)
2. [hexo d 的问题](https://www.v2ex.com/t/289610)
3. [hexo 你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)
