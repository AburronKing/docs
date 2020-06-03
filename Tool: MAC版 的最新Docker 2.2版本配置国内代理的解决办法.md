用过docker的都知道，在pull镜像的时候，使用默认的仓库下载，是十分缓慢的，国内提供了很多的代理镜像站点，你可以使用。你要是查资料的话，全部都是老版的内容，如下：

![](https://pic2.zhimg.com/80/v2-9fb501342dc7b3631916179a07c67e65_1440w.jpg)

让你在什么Daemon中，加入一些国内镜像。

但是新版的是这样的：

![](https://pic3.zhimg.com/80/v2-293d56f666455a8532e70a310c4ed472_1440w.jpg)

几乎查遍全网也没找到新版的介绍，然后开始琢磨。。。

#### 接下来开始操作：

1. 点击 Docker Engine

如图：

![](https://pic3.zhimg.com/80/v2-cc43899115b1a19eb8c3f074cd2d5036_1440w.jpg)

在输入框中加上

```
"registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn", // 中国某高等大学的
    "https://hub-mirror.c.163.com" // 网易的
  ]
```

2. 点击按钮 Apply & Restart 重启即可

3. 这时你可以在终端输入命令

```
docker info
```

看到这个内容的时候，就表示代理成功了！

![](https://pic3.zhimg.com/80/v2-02be43fbe21fd1fa97977c5e41579e5a_1440w.jpg)

### 快去试试吧！