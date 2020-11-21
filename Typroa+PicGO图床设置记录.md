# Typroa+PicGO图床设置记录
> Github/Gitee设置图床记录

[CSDN_PicGo下载链接](https://download.csdn.net/download/Aichilubiantan/12930458)

[Typroa下载链接](https://typora.io/)

## 使用Github:

###  GitHub的设置

1. **登录Github新建仓库**

<img src="https://gitee.com/pengjae/pic/raw/master/img/picgo_github_newrep.png" alt="image-20201117212130628" style="zoom:5%;float:left" />

2. **新建仓库**

<img src="https://gitee.com/pengjae/pic/raw/master/img/pic_github_createrep.png" alt="image-20201117212609446" style="zoom:50%;float:left" />

3. **点击Setting,去Develop setting获取Token**

   <img src="https://gitee.com/pengjae/pic/raw/master/img/pic_gtihub_setting.png" alt="image-20201117212951830" style="zoom:50%;float:left" />

   <img src="C:\Users\25086\AppData\Roaming\Typora\typora-user-images\image-20201117213206767.png" alt="image-20201117213206767" style="zoom:50%;" />

   4.**新建token,记得把repo勾上.拉到最下面,生成token**

   <img src="https://gitee.com/pengjae/pic/raw/master/img/picgo_github_token.png" alt="image-20201117213435212" style="zoom:50%;float:left" />

   5.**将生成的token保存,一会就消失啦**

   ![image-20201117213548548](https://img-blog.csdnimg.cn/img_convert/9a142d8cb5b1c2b9db2ac63605c156ab.png)

   ### PicGo设置

   <img src="https://gitee.com/pengjae/pic/raw/master/img/picgoSetting.png" alt="image-20201117213915127" style="zoom:50%;" />

**设定仓库名:**"用户名/仓库名"

   **设定分支名:** 现在默认的都是**main**,如果有需要可以在GitHub中自行更改成master

   **设定Token:** 将刚才的token复制进去

   **指定存储路径:** 给图片有个文件夹可以放（img/xxx.jpg）

   

   **Server保持默认即可**

   <img src="https://gitee.com/pengjae/pic/raw/master/img/image-20201117214226215.png" alt="image-20201117214226215" style="zoom:30%;" />

   <img src="https://gitee.com/pengjae/pic/raw/master/img/image-20201117214300344.png" alt="image-20201117214300344" style="zoom:33%;" />

   ### 可能遇到的问题:

   1. 如果发现自己的图片看不了了

      ​	1. dns污染:[解决办法](https://blog.csdn.net/dplovel/article/details/107356603)
      
   2. 上传失败原因:
      1. 仓库内文件不允许重名
      2. 服务器原因

   ### Typroa测试

   <img src="https://gitee.com/pengjae/pic/raw/master/img/image-20201117214655936.png" alt="image-20201117214655936" style="zoom:50%;" />

   **点击验证图片上传选项即可验证是否成功**

 >   tips:如果点击第二次验证的话,由于仓库内有相同文件名的文件存在,是会失败的.去github中删除即可
---
## 使用Gitee:
>**想要使用Gitee作为Picgo的默认图床需要安装插件**

### Gitee设置
1. 新建仓库
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111813395342.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FpY2hpbHViaWFudGFu,size_16,color_FFFFFF,t_70#pic_center)
2. Gitee仓库设置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118134357619.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FpY2hpbHViaWFudGFu,size_16,color_FFFFFF,t_70#pic_center)
3. 点击设置，创建Token去啦！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118134518185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FpY2hpbHViaWFudGFu,size_16,color_FFFFFF,t_70#pic_center)
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118134539843.png#pic_center)
在这里插入图片描述




**1. 首先要安装并配置node.js**
[nodejs官网下载](https://nodejs.org/en/)
> 下载完记得配置好环境这些

**2. 安装配置完成后，进入Pigo搜索插件**
> 一定要安装第一个啊，后面那个问题挺多，反正用着用着就不能用了，也没去细究为什么。第一个够用。

![插件里面搜索Gitee](https://img-blog.csdnimg.cn/20201118132831914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FpY2hpbHViaWFudGFu,size_16,color_FFFFFF,t_70#pic_center)
**3. 安装完重启picgo，在Picgo设置中开启Gitee图床**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118133417353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FpY2hpbHViaWFudGFu,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020111813482370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FpY2hpbHViaWFudGFu,size_16,color_FFFFFF,t_70#pic_center)
**4. 生成Token后记得保存！**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118134921194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FpY2hpbHViaWFudGFu,size_16,color_FFFFFF,t_70#pic_center)
### PicGo设置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201118135151817.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0FpY2hpbHViaWFudGFu,size_16,color_FFFFFF,t_70#pic_center)
**按照刚才的方法进行测试即可**
## 使用体验总结：
总的来说还是gitee更舒服，虽然要安装node.js,安装一个插件，但是架不住比GitHub好使。github还容易抽风，时不时图片就读取失败了。
