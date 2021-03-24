# 约束布局(ConstraintLayout)

## 约束规则:

+ 每个识图都必须至少有两个约束条件:一个水平约束,一个垂直约束
+ 只能在共用同一平面的约束手柄与定位点之间创建约束。因此，使徒的垂直平面(左侧和右侧)只能约束在另一个垂直平面上；而基准线则只能约束到其他基准线上。
+ 每个约束句柄只能用于一个约束条件，但是可以在同一定位点上创建多个条件约束（从不同的视图）

### 约束1例子：直接拖拽进来，没有任何约束：

1. 对约束1的理解：在左边的布局树中就会报错：

   `This view is not constrained. It only has designtime positions, so it will jump to (0,0) at runtime unless you add the constraints`

   意思就是说：这个view没有约束，现在这个只是设计时的位置，在运行时就会跳到（0，0）这个位置，也就是屏幕的左上角。

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201209083917.png" alt="image-20201209083910123" style="zoom: 33%; float: left;" />![image-20201209084045566](https://gitee.com/pengjae/pic/raw/master/img/20201209084045.png)<img src="https://gitee.com/pengjae/pic/raw/master/img/20201209084252.png" alt="image-20201209084252173" style="zoom:50%;" />







想要一定摆放在这个位置方法：

1. 自己拉线
2. 使用<img src="https://gitee.com/pengjae/pic/raw/master/img/20201209084442.png" alt="image-20201209084442942" style="zoom:33%;" />
   1. 预测约束器，点击控件后，点击这个按钮，就会自己产生约束条件
      1. <img src="https://gitee.com/pengjae/pic/raw/master/img/20201209084603.png" alt="image-20201209084603909" style="zoom:53%;float:left" />

### 约束2例子：

只能对同一个方向进行约束，不能上下的点去约束左右的点。**只能约束同一水平面的点**	

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201209085114.png" alt="image-20201209085114124" style="zoom:80%;float:left" /><img src="https://gitee.com/pengjae/pic/raw/master/img/20201209085152.png" alt="image-20201209085152129" style="zoom: 50%;" /><img src="https://gitee.com/pengjae/pic/raw/master/img/20201209085408.png" alt="image-20201209085408425" style="zoom:80%;" />

### 约束3例子：

约束句柄（点）只能对一个地方进行约束，不能用这个句柄既约束一个控件，又用这个句柄约束另一个控件。但是一个句柄可以**被**多个控件约束。

