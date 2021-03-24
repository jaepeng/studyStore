# Bitmap

## 基本概念 

1. Bitmap是Android系统中的图像处理的最重要类之一
2. 通过Bitmap我们可以获取图片的信息
3. 获取信息后,可以对其进行缩放,裁剪

类比File类

### 加载方式

![image-20201210074527484](https://gitee.com/pengjae/pic/raw/master/img/20201210074534.png)

#### 为什么使用BitmapFactory来创建Bitmap?

因为Bitmap是非常占用内存的,所以用一个工厂类来创建.

### 为什么要高效加载Bitmap

现在的图片大小都非常大,如果直接加载到内存中非常不好

1. 防止内存溢出
2. 尽可能的节省内存开销
3. 使我们的应用跑的更流畅

### 如何高效加载Bitmap

BitmapFactory.Options,它有以下几个重要属性

1. inJustDecodeBounds:布尔型变量,置为true,不会真正返回一个Bitmap,而是返回bitmap的width和height.并且将这两个属性存放到outWidth&outHeight

2. outWidth&outHeight

3. inSampleSize:采样率

   1. 如果为1,则代表原来大小
   2. 如果为4,则宽高为原来的1/4,像素点为原来的1/16.
   3. 这个数必须是2的幂次的数,如果不是,其他任何值将四舍五入到最接近的2的幂。

   ```java
   package com.example.bitmaptest;
   
   import android.graphics.Bitmap;
   import android.graphics.BitmapFactory;
   import android.os.health.PidHealthStats;
   
   public class BitmapUtil {
       public static Bitmap ratio(String filepath,int piexlW,int piexlh){
           BitmapFactory.Options newOptions=new BitmapFactory.Options();
           newOptions.inJustDecodeBounds=true;
           newOptions.inPreferredConfig=Bitmap.Config.RGB_565;//位深度
           //预加载
           BitmapFactory.decodeFile(filepath,newOptions);
           int originalW=newOptions.outWidth;
           int originalH=newOptions.outHeight;
   
           newOptions.inSampleSize=getSimpleSize(originalW,originalH,piexlW,piexlh);
           //真正的加载
           newOptions.inJustDecodeBounds=false;//一定要记得置为false
           return BitmapFactory.decodeFile(filepath,newOptions);
       }
   
       private static int getSimpleSize(int originalW, int originalH, int piexlW, int piexlh) {
           //如何计算采样率
           int simpleSize=1;//默认不压缩
           if (originalW>originalH&& originalW>piexlW){
               //以宽度计算采样值
               simpleSize=originalW/piexlW;
           }else if (originalW<originalH&& originalH> piexlh){
               //以高度计算采样值
               simpleSize=originalH/piexlh;
           }
           if (simpleSize<=0){
               //保证正确
               simpleSize=1;
           }
           return simpleSize;
       }
   }
   ```

   ## Android 缓存策略

   ### 定义：

   缓存就是将从服务器请求到的数据（Json，File）等保存到本地，这就是缓存。相对于内存而言。

   ### 缓存优势

   1. 对一些不是经常发送的数据，直接使用本地缓存，提升应用响应速度
   2. 不再频繁的请求服务，可以降低服务器的负载压力。
   3. 离线阅读等

   ### 常见使用场景

   1. 对Bitmap和File等数据进行缓存，无需每次都从服务器下载。
   2. 数据更新不需要实时更新，采用缓存机制

   ### 常用缓存策略

   1. **LruCache**实现内存缓存
   2. **DiskLruCache**实现硬盘缓存
   3. SQLite实现缓存（主要是持久化数据，不是缓存的重点，是一种思路）

   #### [LruCache](https://blog.csdn.net/lj402159806/article/details/104116446)

   ##### 概念

   1. 近期最少使用算法
   2. 内部采用LinkedHashMap——最近使用的对象用强引用存储，如果是刚访问的就放在尾部
   3. 为了取代SoftReference

   ##### 为什么是[LinkedHashMap](https://blog.csdn.net/lj402159806/article/details/104064284)？

   - LinkedHashMap基于HashMap，具有HashMap高效查找、自动扩容等特性
   - 在HashMap基础上，增加了一个双向链表存储K-V对、实现了自己的遍历器LinkedHashIterator，默认情况下可以做到根据数据插入顺序有序地遍历
   - 提供重载构造方法供外部控制accessOrder，以实现根据访问顺序有序地遍历[accessOrder详解](https://blog.csdn.net/qq_35634181/article/details/103833875)
     - 所以我们就清楚了当accessOrder设置为false时，会按照插入顺序进行排序，**当accessOrder为true时，会按照访问顺序**
   

   
#### 注意问题：
   
1. 内存缓存使用的还是内存，所以要动态调整大小
   2. 注意版本适配
   
#### 代码用法
   
```java
   package com.example.bitmaptest;
   
   import android.graphics.Bitmap;
   import android.util.LruCache;
   import android.widget.ImageView;
   
   public class SimpleImageLoader {
       private static SimpleImageLoader mLoader;
       private LruCache<String, Bitmap> mLruCache;
       private static SimpleImageLoader getInstance(){
           if (mLoader==null){
               synchronized (SimpleImageLoader.class){
                   if (mLoader==null){
                       mLoader=new SimpleImageLoader();
                   }
               }
           }
           return mLoader;
       }
       //私有构造方法：初始化缓存对象
       private SimpleImageLoader(){
           int maxSize= (int) (Runtime.getRuntime().maxMemory()/8);//最大缓存大小
           mLruCache=new LruCache<String,Bitmap>(maxSize){
               @Override
               protected int sizeOf(String key, Bitmap value) {
                   return value.getByteCount();
               }
           };
   
       }
       /**
        * 加载网络图片
        */
       public void displayImage(ImageView view,String url){
           //判断缓存中是否有这张图片
           Bitmap bitmap=getBitmapFromCache(url);
           if (bitmap!=null){
               view.setImageBitmap(bitmap);
           }else{
               downloadImage(view,url);
           }
       }
   
       /**
        * 从缓存中读取图片
        * @param url
        * @return
        */
       private Bitmap getBitmapFromCache(String url){
           return mLruCache.get(url);
       }
   
       /**
        * 将下载下来的图片保存到缓存中
        * @param bitmap
        * @param url
        */
       private void putBitmapToCache(Bitmap bitmap,String url){
           if (bitmap!=null){
   
               mLruCache.put(url,bitmap);
           }
   
       }
   
       /**
        * 下载图片并保存到内存中
        * @param imageView
        * @param url
        */
       private void downloadImage(final ImageView imageView,final String url){
           //Okhttp从网上下载图片然后加载到ImageVie中
   
       }
   }
   ```
   
### DiskLruCache
   
1. 非常方便的将我们数据缓存到本地空间
   
#### 使用：
   
![image-20201210094830852](https://gitee.com/pengjae/pic/raw/master/img/20201210094830.png)
   
#### 注意问题
   
![image-20201210094942388](https://gitee.com/pengjae/pic/raw/master/img/20201210094942.png)
   

   
#### 使用
   
1. 使用观察者模式封装这种第三方接口
   2. 封装后，降低了程序和第三方库的耦合
   3. 如果以后出来一个更方便的第三方库，那么就只用修改我们工具类的方法就行了。
   
