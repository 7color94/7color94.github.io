---
layout: post
title: Shadowsocks
categories:
- Programming
tags:
- shadowsocks
---

针对GFW，渴求真理和自由的程序员绞尽脑汁，创造出许多梯子工具。我在大二的时候尝试过一些翻墙软件，用过免费VPN，用过GoAgent等等，这些都被封杀得很快，直到后来了解到了Shadowsocks。（提到Shadowsocks，大家一定不陌生这句话：**“Removed according to regulations.”**）

在我写这篇文章之前，已经使用Shadowsocks快一年了，见证了ss从未封杀到被封杀。写这篇文章，是因为：

- 1.翻墙对我很重要，我要将“Shadowsocks科学上网”的方案备份下来，我的生活已离不开翻墙
- 2.这几天很多人问我借ss账号，我有必要告诉他人如何科学上网

说了那么多，我其实并不会在这篇介绍如何利用shadowsocks科学上网，因为这些教程随意Google一大堆（当然，Baidu是不可以的，因为根据相关法律......）。那么，我既然不能翻墙，我怎么google，这不是一个死循环吗？那我直接就贴出一篇教程来：

[FindSpace：SHADOWSOCKS科学上网](http://www.findspace.name/res/956)

FindeSpace是我的博客友链，我当时也是因为搜索到了Findspace的教程，搞定了ss，和他交了朋友

所以当时按照他的教程：

我在host1plus上买了一个最最便宜的vps（美国洛杉矶节点），一个月10+RMB，然后在vps上装了ubuntu系统，ssh上去安装了python和shadowsocks（pip install），最后编写了json格式的配置文件，并使用nohup ssserver命令在后台启动服务。 最后在本地也同样安装shadowsocks，以及编写配置文件，使用sslocal启动服务。 我是为了翻墙才买的vps，所以，最终用firefox代理插件，配置本地的shadowsocks代理端口，即可科学上网了（现在换了MAC，不需要任何代理插件了，也不需要命令行启动ss客户端了）。 需要说明的是，ios和android都是有shadowsocks客户端的（只不过ios上的是收费软件，叫做shadowrocket），所以，科学上网不仅限于PC机端。还有，买个vps跑个shadowsocks，未免太浪费了，还可以跑跑其他程序的，很多人都用vps建站搭博客等。

如果你不熟悉简单的shell命令，也不想那么烦神去搭什么ss服务，那么可以搜索牛叉网络加速（朋友推荐，听他说还不错），去上面直接买一个账号吧

再给大家列举一些资源列表：

- [long-live-shadowsocks](https://github.com/Long-live-shadowsocks)：仅仅用来备份 shadowsocks 相关项目（re-uploaded because I can）

这个仓库很强大，ios，android，mac，win的ss客户端都有，快去下载吧

Best Wishes !!