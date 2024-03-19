---
title: Embeddable Python在工程实践上的使用
description: ' '
categories: 技术
abbrlink: f092de32
date: 2023-10-02 11:02:00
---

在维护 AstrBot 项目时，发现很多人电脑上的 Python 版本与项目要求的 3.9+ 不一致，在一开始，我是写了一个启动脚本，自动在 [npm.taobao.org](http://npm.taobao.org) 下载合适的 Python 版本的 exe 文件。不过在最近越来越多的人反映中发现了一些问题：

1. Windows 下，安装了 Python 之后，由于 Windows 的一些机制，环境变量（PATH）需要重启终端或重启电脑之后才生效，一些人不知道这一点，也很难广泛科普。
2. Windows下，一些人之前有安装过 Python，但是版本不合适，程序装了合适版本的 Python 后，又由于环境变量设置的问题，导致 `python` 或 `python3` 指令还是链接到原来的 Python 上。

于是我就考虑开始使用 Embbed Python，内置一个 Python，这样就行了嘛orz。

![](/images/20231002112107815/20231002112146200.png)


在 [python.org](http://python.org) 上，是有提供 Embeddable Python 的。并且 [npm.taobao.org](http://npm.taobao.org) 等国内镜像也有对应的文件，不用担心用户的梯子问题。由于下载下来是一个压缩包，需要写代码自行解压。只需要使用 python下的 `zipfile` 库即可解压该文件。

于是我就发现了问题：

```bash
PS D:\Project\QQChatGPTLauncher\dist> .\python\python.exe -m pip
D:\Project\QQChatGPTLauncher\dist\python\python.exe: No module named pip
PS D:\Project\QQChatGPTLauncher\dist>
```

默认情况下是没有 pip 库的。这个倒是容易解决，于是我下载并使用 `get-pip.py` 来安装。

```bash
PS D:\Project\QQChatGPTLauncher\dist> .\python\python.exe get-pip.py
...(ignore)
PS D:\Project\QQChatGPTLauncher\dist> .\python\python.exe -m pip
D:\Project\QQChatGPTLauncher\dist\python\python.exe: No module named pip
```

安装成功后依然报错，

在寻找了国内的网页后，没发现解决方案，于是转向国外搜索。在一篇相关的文章下，找到了这个：

```bash

Even if explicitly stated that the embeddable version of Python does not support Pip, it is possible with care. You need to:

Download and unzip Python embeddable zip file.

In the file python39._pth or similar, uncomment the import command. Result should look similar to this:

python39.zip
.
import site
```

于是我试着将 python310._pth 内的 `import site` 的注释去掉，

![](/images/20231002112107815/20231002112159905.png)

然后就可以用了。可喜可贺。

接下来又发现了问题：

使用了这个内置的Python之后，没办法 import 自己工程下的库。

于是试着修改 python310._pth 文件，里面加上

```bash
../QQChannelChatGPT
```

也就是以 python310._pth 所在目录下，以相对路径的方式把自己的工程项目放进去。

再次重试，完美运行。

那么， python310._pth 是什么呢？

Python在遍历已知的库文件目录过程中，如果见到一个._pth 文件，**就会将文件中所记录的路径加入到 sys.path 设置中**，于是 .pth 文件说指明的库也就可以被 Python 运行环境找到了。

让我们来看看 `import site` 会得到什么

![](/images/20231002112107815/20231002112209274.png)

好吧，什么都没输出捏，还是看看源代码吧：

**`site`** 模块会根据系统的配置和环境变量，修改 **`sys.path`**，即 Python 模块搜索路径。这可以确保 Python 解释器能够找到正确的模块和库。