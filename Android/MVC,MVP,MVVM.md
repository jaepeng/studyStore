# MVC,MVP,MVVM

[Android三种框架的比较——教你认清MVC，MVP和MVVM_xiaoyugehang的博客-CSDN博客_android mvp和mvvm的区别](https://blog.csdn.net/qq_42442129/article/details/80736265)

## MVC

![mvc](https://gitee.com/pengjae/pic/raw/master/img/20201201083543.png)

### M:model V:View C: Controller

+ Model:JavaBean
+ View:Xml
+ Controller:Activity

### MVC工作原理:

比如你的界面有一个按钮，按下这个按钮去网络上下载一个文件，这个按钮是view层的，是使用xml来写的，而那些和网络连接相关的代码写在其他类里，比如你可以写一个专门的networkHelper类，这个就是model层，那怎么连接这两层呢？是通过button.setOnClickListener()这个函数，这个函数就写在了activity中，对应于controller层。

###  劣势:

+ xml作为view层,控制能力实在太弱,当想要要动态改变一个页面北京,或者动态隐藏\显示一个按钮,都没法再Xml中做,而是要在Activity中.这就让作为Controller层的Activity的职责混乱了。
+ View和Model层是相互可知的，意味着两层之间存在耦合，耦合对于一个大型程序来说非常致命

## MVP

![mvp](https://gitee.com/pengjae/pic/raw/master/img/20201201083648.png)

### M:Model V:View P:Presenter

+ Model:JavaBean
+ View Activity
+ Presenter：接口实现类

### 优势

+ view层和model层不在相互告知，完全解耦。Presenter层充当了桥梁的作用
+ 用于操作的View层发出的事件传递到persenter层中，presenter层去操作model层，并且将数据返回给View层。整个过程中View层和model层没有联系。
+ 虽然图上看着好像presenter层和View层耦合了，实际上他们之间通过接口进行交流的。而不是直接交流。
+ 当然，其实最好的方式是使用fragment作为view层，而activity则是用于创建view层(fragment)和presenter层(presenter)的一个控制器。

## MVVM

![mvvm](https://gitee.com/pengjae/pic/raw/master/img/20201201084735.png)

> 从图中看出，它和MVP的区别貌似不大，只不过是presenter层换成了viewmodel层，还有一点就是view层和viewmodel层是相互绑定的关系，这意味着当你更新viewmodel层的数据的时候，view层会相应的变动ui。

