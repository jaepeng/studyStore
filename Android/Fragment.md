# Fragment

[参考链接](https://www.jianshu.com/p/9f538c3a1918)

[启舰参考链接](https://blog.csdn.net/harvic880925/article/details/44927375)

## Framgent的作用

1. 同样的界面Activity占用内存比Fragment要多，响应速度Fragment比Activty在中低端手机上快了很多，甚至能达到好几倍
2. 易于移植平板

## Fragment的优点

1. Fragment可以使你能够将activity分离成多个可重用的组件，每个都有它自己的生命周期和UI。
2. Fragment可以轻松得创建动态灵活的UI设计，可以适应于不同的屏幕尺寸。从手机到平板电脑。
3. Fragment是一个独立的模块,紧紧地与activity绑定在一起。可以运行中动态地移除、加入、交换等。
4. Fragment提供一个新的方式让你在不同的安卓设备上统一你的UI。
5. Fragment 解决Activity间的切换不流畅，轻量切换。
6. Fragment 替代TabActivity做导航，性能更好。
7. Fragment 在4.2.版本中新增嵌套fragmeng使用方法，能够生成更好的界面效果。
8. Fragment做局部内容更新更方便，原来为了到达这一点要把多个布局放到一个activity里面，现在可以用多Fragment来代替，只有在需要的时候才加载Fragment，提高性能

## Fragment生命周期

![img](https://gitee.com/pengjae/pic/raw/master/img/20201124215553.webp)

> **onAttach**：onAttach()在fragment与Activity关联之后调调查用。需要**注意**的是，初始化fragment参数可以从getArguments()获得，但是，当Fragment附加到Activity之后，就无法再调用setArguments()。所以除了在最开始时，其它时间都无法向初始化参数添加内容

> **onCreate**：fragment初次创建时调用。尽管它看起来像是Activity的OnCreate()函数，但这个只是用来创建Fragment的。此时的Activity还没有创建完成，因为我们的Fragment也是Activity创建的一部分。所以如果你想在这里使用Activity中的一些资源，将会获取不到。比如：获取同一个Activity中其它Frament的控件实例。(代码如下：)，如果想要获得Activity相关联的资源，必须在onActivityCreated中获取。

> **onCreateView**：在这个fragment构造它的用户接口视图(即布局)时调用。
>
> **onViewCreated()**:在onCreateVIew调用后立即调用

> **onActivityCreated**：在Activity的**OnCreate()结束后**，会调用此方法。所以到这里的时候，Activity已经创建完成！在这个函数中才可以使用Activity的所有资源。如果把下面的代码放在这里，获取到的btn_Try的值将不会再是空的！

> **onStart**：当到OnStart()时，Fragment对用户就是可见的了。但用户还未开始与Fragment交互。在生命周期中也可以看到Fragment的OnStart()过程与Activity的OnStart()过程是绑定的。意义即是一样的。以前你写在Activity的OnStart()中来处理的代码，用Fragment来实现时，依然可以放在OnStart()中来处理。

> **onResume**：当这个fragment对用户可见并且正在运行时调用。这是Fragment与用户交互之前的最后一个回调。从生命周期对比中，可以看到，Fragment的OnResume与Activity的OnResume是相互绑定的，意义是一样的。它依赖于包含它的activity的Activity.onResume。当OnResume()结束后，就可以正式与用户交互了。

> **onPause**：此回调与Activity的OnPause()相绑定，与Activity的OnPause()意义一样。

> **onStop**：这个回调与Activity的OnStop()相绑定，意义一样。已停止的Fragment**可以直接返回到OnStart()回调**，然后调用OnResume()。而Activiy如果需要从不可见状态回到可见状态的话的话需要调用onRestart才能进入到onStart();

> **onDestroyView**：如果Fragment即将被结束或保存，那么撤销方向上的下一个回调将是onDestoryView()。会将在onCreateView创建的视图与这个fragment分离。下次这个fragment若要显示，那么将会创建新视图。这会在onStop之后和onDestroy之前调用。这个方法的调用同onCreateView是否返回非null视图无关。它会潜在的在这个视图状态被保存之后以及它被它的父视图回收之前调用。

> **onDestroy**：当这个fragment不再使用时调用。需要注意的是，它即使经过了onDestroy()阶段，但仍然能从Activity中找到，因为它还没有Detach。

> **onDetach**：Fragment生命周期中最后一个回调是onDetach()。调用它以后，Fragment就不再与Activity相绑定，它也不再拥有视图层次结构，它的所有资源都将被释放。

## 创建Fragment

### 静态创建Fragment

![img](https://gitee.com/pengjae/pic/raw/master/img/20201124215619.webp)



### 动态创建Fragment

#### ![img](https://gitee.com/pengjae/pic/raw/master/img/20201124215634.webp)Replace Fragment

```java
//使用replace的方式，用Fragment2,代替Frgment1(Fragment1已经被添加了)
 Fragment2 f2=new Fragment2();
FragmentManager fragmentManager=getSupportFragmentManager();
FragmentTransaction transaction = fragmentManager.beginTransaction();
transaction.replace(R.id.fragment_container,f2);
transaction.commit();
```

##### 生命周期表现：

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201125144616.png" alt="image-20201125144606979" style="zoom:50%;" />

##### 分析

**这里的三个划线处需要注意：**

1. 当使用replace进行替换时，Fragment2直接调用onAttach和onCreate。然后才轮到Fragment1的消亡
2. 当Fragment1最后消亡后(onDeatch)后Fragment1才开始创建View（onCreateView)等工作
3. 如果之前有多个Fragment的话，等进来的Fragment调用了onAttach和onCreate后，其他Fragment进入销毁程序，等其他全部销毁完成，才会继续该Fragment的下一步初始化

**replace方法**的内部是对之前的Fragment进行了删除操作，对当前传入的Fragment进行了添加操作

**Android7.0以前这个方法有个bug**，在fro循环中根据mManager.mAdded.size();这个大小来删除，由于在里面执行了remove操作，所以这个size实际上是越来越小的，因此就会有删除不干净的问题。

#### Add Fragment操作

##### 生命周期表现

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201125144726.png" alt="image-20201125144724618" style="zoom:67%;" />

##### 分析

1. 直接对Fragment2进行添加，并没有调用Fragment1的onPause和onStop方法，为什么这样？因为这两个方法是与Activity的生命周期绑定的
2. 这时候如果在点击按钮添加Fragment1，又会从onAttach()开始，因为每次add都是一个新的Fragment对象
3. 等添加了多个Fragmetn后，这个时候如果使用replce方法就会让之前所有添加的Fragment都销毁。并且是倒序的，因为Fragment的add方法是将Fragment添加到Fragment栈里去，这个时候删除也是出栈删除,所以先来的后删除.
4. 下面这张图是点击replace按钮,将Fragment2replace上来,可以看到,Fragment2执行到**onCreate**后就停下来等其他在栈中的Fragment删除后,才继续onCreateView等操作.

![image-20201125150239222](https://gitee.com/pengjae/pic/raw/master/img/20201125150241.png)

![image-20201125145958095](https://gitee.com/pengjae/pic/raw/master/img/20201125150000.png)

![img](https://raw.githubusercontent.com/jaepeng/myPicGo/main/img/Fragment_add_remove_stack)



### 动态添加的步骤

1. 获取到FragmentManager，在V4包中通过getSupportFragmentManager，在系统中原生的Fragment是通过getFragmentManager获得的。
2. 开启一个事务，通过调用beginTransaction方法开启。
3. 向容器内加入Fragment，一般使用add或者replace方法实现，需要传入容器的id和Fragment的实例。
4. 提交事务，调用commit方法提交。

## Fragment管理与Fragment事务

![img](https://gitee.com/pengjae/pic/raw/master/img/20201125145032.webp)

## Fragment和Activity交互

![img](https://upload-images.jianshu.io/upload_images/7508328-479043c8a296ec8d.png?imageMogr2/auto-orient/strip|imageView2/2/w/920/format/webp)

### 组件获取

| Fragment获得Activity中的组件,要在onActivityCreated()之后 | getActivity().findViewById(R.id.list)；              |
| -------------------------------------------------------- | ---------------------------------------------------- |
| Activity获得Fragment中的组件(根据id和tag都可以)          | getFragmentManager.findFragmentByid(R.id.fragment1); |

### 数据获取

1. **Activit传递数据给Fragment:**

   在Activity中创建Bundle数据包,调用Fragment实例的setArguments(bundle) 从而将Bundle数据包传给Fragment,然后Fragment中调用getArguments获得 Bundle对象,然后进行解析就可以了

   **step1:设置bundle,并且进行传递**

   ```java
   mBundle.putString("container","container text");
   f1.setArguments(mBundle);
   mTransaction.add(R.id.fragment_container,f1,"20201110");//最后这个tag是一个标识符,可以不加
   ```

   **step2:在Fragment中获取Activity传来的数据**

   ```java
   Bundle arguments = this.getArguments();
   String container1 = arguments.getString("container");
   ```

2. **Fragment传递数据给Activity**

   在Fragment中定义一个内部回调接口,再让包含该Fragment的Activity实现该回调接口, Fragment就可以通过回调接口传数据了

   **Step 1:定义一个回调接口:(Fragment中)**

   ```java
   /*接口*/  
   public interface CallBack{  
       /*定义一个获取信息的方法*/  
       public void getResult(String result);  
   }  
   ```

   **Step 2：接口回调（Fragment中）**

   ```java
   /*接口回调*/  
   public void getData(CallBack callBack){  
       /*获取文本框的信息,当然你也可以传其他类型的参数,看需求咯*/  
       String msg = editText.getText().toString();  
       callBack.getResult(msg);  
   } 
   ```

   **Step 3:使用接口回调方法读数据(Activity中)**

   ```java
   /* 使用接口回调的方法获取数据 */  
   leftFragment.getData(new CallBack() {  
    		@Override  
          public void getResult(String result) {              /*打印信息*/  
             Toast.makeText(MainActivity.this, "-->>" + result, 1).show();  
          }
   }); 
   ```

3. **Fragment与Fragment之间的数据互传**(replace时传递)

   **step1:在Fragment2中写一个方法使得调起Fragment2时可以拿到数据**

   ```java
    public static Fragment2 newInstance(String text) {
          Fragment2 fragment = new Fragment2();
          Bundle args = new Bundle();
          args.putString("param", text);
          fragment.setArguments(args);
          return fragment;
   }
   ```

   step2:在Fragment1调起Fragment2

   ```java
   public void onClick(final View view) {
          Fragment2 fragment2 = Fragment2.newInstance("从Fragment1传来的参数");
          FragmentTransaction transaction = getFragmentManager().beginTransaction();
          transaction.add(R.id.main_layout, fragment2);
          transaction.addToBackStack(null);
          transaction.commit();
   }
   ```

   step3:在Fragment2中接收参数

   ```java
   public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
           View view =  inflater.inflate(R.layout.fragment2, container, false);
           if (getArguments() != null) {
               String mParam1 = getArguments().getString("param");
               TextView tv =  (TextView)view.findViewById(R.id.textview);
               tv.setText(mParam1);
           }
           return view;
   }
   ```

4. **同一个Container下,”左右两边”的Fragment传递数据**

   1. 在Activity中进行操作

      1. 拿到两个Fragment的控件
      2. 获取数据进行展示即可(例如点击list中的一个位置,将该位置的信息传递给另一个Fragment的控件进行展示)

   2. 在Fragment中进行操作

      step1:获取控件下问再详细列出方法

      step2:拿到对应的控件后,设置点击事件,进行数据的获取.

      step:对数据进行展示

   

5. **获取控件**

   1. 获取自己Fragment的控件

      1. 方法一:在onCreateView中获取

      ```java
      public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
         View view=inflater.inflate(R.layout.fragment1,container,false);
         TextView textView=view.findViewById(R.id.f1_tv);
          return view;
      }
      ```

      **tips:**由于在onCreateView()中，还没有创建视图，所以在这里如果使用getView()方法将返回空。所以如果要获取其实图中指定控件的引用，只用用inflater.inflate()返回的rootView；在这个rootView中用findViewById来查找。

      2. 方法二:在onActivityCreated()函数中获取。此时的Activiy已经创建完毕,那么其中的Fragment自然也就创建完毕了

      ```java
         @Override
          public void onActivityCreated(@Nullable Bundle savedInstanceState) {
              super.onActivityCreated(savedInstanceState);
              TextView textView=getView().findViewById(R.id.f1_tv);
          }
      ```

   2. 获取别的别的Fragment的控件。一定要在onActivityCreated()方法中获取，因为只有这个时候Activity资源全部加载完毕。

      ```java
      public void onActivityCreated(Bundle savedInstanceState) {
      super.onActivityCreated(savedInstanceState);
      mFragment2_tv = (TextView) getActivity().findViewById(R.id.fragment2_tv);//获取其它fragment中的控件引用的唯一方法!!!
      
      }
      ```

      **总结:**既然都可以在onActivityCreated()方法中获取,那么都在这里进行获取,并且在这里进行数据操作即可.

   5. **在各自的Fragment中进行设置,涉及到两个Fragment间进行数据传递**
      1. 由于是即时的,点击FragmentA,B就要得到数据,所以需要一个媒介为它们两个传递数据,这个媒介就是Activity
      2. 需要先将Fragment1中的数据通过回调传给Activity
      3. 再将Acivity中的数据设置给Fragment2的控件上
      4. [参考链接](https://blog.csdn.net/harvic880925/article/details/44966913)

   ## Fragmetn的管理

   [参考链接](https://blog.csdn.net/harvic880925/article/details/44927375)

   [参考链接二](https://blog.csdn.net/harvic880925/article/details/44948027)

   ### 1.FragmentManager

   要管理activity中的fragments，你就需要使用FragmentManager。通过getFragmentManager（）或getSupportFragmentManager（）获得
   
   **常用的方法有：**
   
   ```java
manager.findFragmentById();  //根据ID来找到对应的Fragment实例，主要用在静态添加fragment的布局中，因为静态添加的fragment才会有ID
   manager.findFragmentByTag();//根据TAG找到对应的Fragment实例，主要用于在动态添加的fragment中，根据TAG来找到fragment实例
manager.getFragments();//获取所有被ADD进Activity中的Fragment
   ```
   
   ### FragmentTransaction
   
   ```java
   //将一个fragment实例添加到Activity的最上层
   add(int containerViewId, Fragment fragment, String tag);
   //将一个fragment实例从Activity的fragment队列中删除
remove(Fragment fragment);
   //替换containerViewId中的fragment实例，注意，它首先把containerViewId中所有fragment删除，然后再add进去当前的fragment
   replace(int containerViewId, Fragment fragment);
   ```
   
   #### 1、FragmentTransaction事务回滚使用方法：
   
   1.以事务为单位