# Typora Windows下修改快捷键

> Windows情况下：
>
> Typoar版本：0.9.96(beta)

这次我主要是想文件树快捷键变成更符合我习惯的“ctrl+shift+2".查看了官方文档后，将这个步骤记录下来。

1. 打开偏好设置，点击里面的打开高级设置

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201202075548.png" alt="image-20201202075548853" style="zoom:50%;" />

2. 进入文件夹，打开[conf.user.json]

   + 如果没有该文件的话，创建一个即可。

   <img src="https://gitee.com/pengjae/pic/raw/master/img/20201202075816.png" alt="image-20201202075816213" style="zoom:50%;" />

3.文本方式打开后，可以看到，有一块keyBinding

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201202080001.png" alt="image-20201202080001241" style="zoom:50%;" />

+ 它是以json文件形式存在的,修改时要符合json语法.
+ 找到对应想修改的快捷键名称,例如文件树(File Tree)
+ <img src="https://gitee.com/pengjae/pic/raw/master/img/20201202080448.png" alt="image-20201202080447986" style="zoom:60%;" />
+ 试了一下,好像中英文都可以识别，修改。我个人使用英文作为key来修改。

**保存并重启Typro即可**