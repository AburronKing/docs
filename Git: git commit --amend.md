## git commit --amend 用法详解

有时你提交过代码之后，发现一个地方改错了，你下次提交时不想保留上一次的记录；或者你上一次的commit message的描述有误，这时候你可以使用接下来的这个命令：git commit --amend。

git功能十分强大，接下来我将讲解一下git commit --amend命令的用法~

git log之后，可以看到你之前提交过的git历史：

![alt pic1](https://pic1.zhimg.com/80/v2-0d0a335a3a87091c40ce138b8be1a9f0_hd.jpg)

接下来，在bash里输入wq退出log状态，执行：

```$ git commit --amend```

这时bash里会出现以下内容：

![alt pic2](https://pic3.zhimg.com/80/v2-b2ab0be08ad6a72d151c5a0ef94ed646_hd.jpg)

其中，second commit 是你上次提交的描述，下面是一下说明信息，有告诉你上次提交的文件信息等等，可忽略。接下来你要是想修改描述信息的话。直接键入：i，此时进入了输入模式，变成这样子：

![alt pic3](https://pic2.zhimg.com/80/v2-c7756d0088e911ef843b5600365926bd_hd.jpg)

可用键盘上下键转到描述所在的那一行，然后进行修改：

![alt pic4](https://pic3.zhimg.com/80/v2-c5da05b7c480adeab60361e7c97c298e_hd.jpg)

修改完成后，按下 Esc键退出编辑模式，在键入 :wq 回车退出并保存修改，完成提交。这是你再git log 看一下提交日志：

![alt pic5](https://pic2.zhimg.com/80/v2-c6e4d7f6508340a3b66f187f0f8580c5_hd.jpg)

已经修改了提交描述信息，且原来的git版本没有了~~~喜大普奔！！你完成了~~

但是有个地方要注意，就是该操作会改变你原来的commit id哦。