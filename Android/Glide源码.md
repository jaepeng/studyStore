# Glide源码

文章出处：[Android 主流开源框架（六）Glide 的执行流程源码解析 (juejin.cn)](https://juejin.cn/post/6844904152636604424)

```java
Glide.with(this).load(url).into(imageView);
```

Glide默认配置了内存与磁盘缓存.为了方便,先进行禁用

```
Glide.with(this)
        .load(url)
        .skipMemoryCache(true) // 禁用内存缓存
        .diskCacheStrategy(DiskCacheStrategy.NONE) // 禁用磁盘缓存
        .into(imageView);
```

## 2.1 with()

```
//6个重载方法
Glide#with(Context context)
Glide#with(Activity activity)
Glide#with(FragmentActivity activity)
Glide#with(Fragment fragment)
Glide#with(android.app.Fragment fragment)
Glide#with(View view)
```

大致分为以下两种情况：

+ Application(Context)
+ 非Application(Activity、Fragment、View、这里的View获取的是它所属的Activity或Fragment)

最后都会去调用：

```java
  /*Glide*/
  public static RequestManager with(@NonNull FragmentActivity activity) {
    return getRetriever(activity).get(activity);
  }
```

```java
//去看getRetriever()方法
  /*Glide*/
  private static RequestManagerRetriever getRetriever(@Nullable Context context) {
	...
    return Glide.get(context).getRequestManagerRetriever();
  }
```

```java
 //Glide.get()方法
//双重校验的方式获得Glide实例
  public static Glide get(@NonNull Context context) {
    if (glide == null) {
      //（1）
      GeneratedAppGlideModule annotationGeneratedModule =
          getAnnotationGeneratedGlideModules(context.getApplicationContext());
      synchronized (Glide.class) {
        if (glide == null) {
          //（2）
          checkAndInitializeGlide(context, annotationGeneratedModule);
        }
      }
    }

    return glide;
  }
```

#### 关注点（1）

```java
  /*Glide*/
  private static GeneratedAppGlideModule getAnnotationGeneratedGlideModules(Context context) {
    GeneratedAppGlideModule result = null;
      Class<GeneratedAppGlideModule> clazz =
          (Class<GeneratedAppGlideModule>)
              Class.forName("com.bumptech.glide.GeneratedAppGlideModuleImpl");
      result =
 clazz.getDeclaredConstructor(Context.class).newInstance(context.getApplicationContext());
 
    ...
    
    return result;
  }

```



该方法用来实例化我们用 @GlideModule 注解标识的自定义模块，这里提一下自定义模块的用法。

+ Glide3

  1. 定义一个类实现 GlideModule，如下：

  ```java
  public class MyGlideModule implements GlideModule {
  
      @Override
      public void applyOptions(Context context, GlideBuilder builder) {
  		//Glide配置
      }
  
      @Override
      public void registerComponents(Context context, Glide glide) {
  		//替换Glide组件
      }
  }
  ```

  其中 **applyOptions()** 与 **registerComponents()** 方法分别用来更改 Glide 的配置以及替换 Glide 组件。

  2. 然后在 AndroidManifest.xml 文件中加入如下配置：

     ```java
         <application>
     
             ...
     
             <meta-data
                 android:name="com.wildma.myapplication.MyGlideModule"
                 android:value="GlideModule" />
         </application>
     ```

+ Glide 4

  1. 定义一个类实现 AppGlideModule，然后加上 @GlideModule 注解即可。如下：

     ```java
     @GlideModule
     public final class MyAppGlideModule extends AppGlideModule {
     
         @Override
         public void applyOptions(@NonNull Context context, @NonNull GlideBuilder builder) {
         }
     
         @Override
         public void registerComponents(@NonNull Context context, @NonNull Glide glide, @NonNull Registry registry) {
         }
     }
     ```

  #### 关注点（2）

  ```java
    /*Glide*/
    private static void checkAndInitializeGlide(
        @NonNull Context context, @Nullable GeneratedAppGlideModule generatedAppGlideModule) {
      // 不能重复初始化
      if (isInitializing) {
        throw new IllegalStateException(
            "You cannot call Glide.get() in registerComponents(),"
                + " use the provided Glide instance instead");
      }
      isInitializing = true;
      initializeGlide(context, generatedAppGlideModule);
      isInitializing = false;
    }
    
      /*Glide*/
      private static void initializeGlide(
        @NonNull Context context, @Nullable GeneratedAppGlideModule generatedAppGlideModule) {
      initializeGlide(context, new GlideBuilder(), generatedAppGlideModule);
    }
    
      /*Glide*/
    private static void initializeGlide(
        @NonNull Context context,
        @NonNull GlideBuilder builder,
        @Nullable GeneratedAppGlideModule annotationGeneratedModule) {
      Context applicationContext = context.getApplicationContext();
      List<com.bumptech.glide.module.GlideModule> manifestModules = Collections.emptyList();
      if (annotationGeneratedModule == null || annotationGeneratedModule.isManifestParsingEnabled()) {
  	  //（1）
        manifestModules = new ManifestParser(applicationContext).parse();
      }
  
  	...
  
  	// 从注解生成的 GeneratedAppGlideModule 中获取 RequestManagerFactory
      RequestManagerRetriever.RequestManagerFactory factory =
          annotationGeneratedModule != null
              ? annotationGeneratedModule.getRequestManagerFactory()
              : null;
  	// 将 RequestManagerFactory 设置到 GlideBuilder
      builder.setRequestManagerFactory(factory);
  	//（2）
      for (com.bumptech.glide.module.GlideModule module : manifestModules) {
        module.applyOptions(applicationContext, builder);
      }
  	//（3）
      if (annotationGeneratedModule != null) {
        annotationGeneratedModule.applyOptions(applicationContext, builder);
      }
  	//（4）
      Glide glide = builder.build(applicationContext);
      for (com.bumptech.glide.module.GlideModule module : manifestModules) {
        try {
  		// (5)
          module.registerComponents(applicationContext, glide, glide.registry);
        } catch (AbstractMethodError e) {
          throw new IllegalStateException(
              "Attempting to register a Glide v3 module. If you see this, you or one of your"
                  + " dependencies may be including Glide v3 even though you're using Glide v4."
                  + " You'll need to find and remove (or update) the offending dependency."
                  + " The v3 module name is: "
                  + module.getClass().getName(),
              e);
        }
      }
      if (annotationGeneratedModule != null) {
  	  // (6)
        annotationGeneratedModule.registerComponents(applicationContext, glide, glide.registry);
      }
  	// 注册组件回调
      applicationContext.registerComponentCallbacks(glide);
  	//（7）
      Glide.glide = glide;
    }
  
  ```

  可以看到，调用 checkAndInitializeGlide() 方法后最终调用了最多参数的 initializeGlide() 方法。源码中我标记了 7 个关注点，分别如下：

  - （1）：将 AndroidManifest.xml 中所有值为 GlideModule 的 meta-data 配置读取出来，并将相应的自定义模块实例化。也就是实例化前面演示的在 Glide 3 中的自定义模块。
  - （2）（3）：分别从两个版本的自定义模块中更改 Glide 的配置。
  - （4）：使用[建造者模式](https://wildma.github.io/blog/364ea8cc.html#toc-heading-14)创建 Glide。
  - （5）（6）：分别从两个版本的自定义模块中替换 Glide 组件。
  - （7）：将创建的 Glide 赋值给 Glide 的静态变量。

  继续看下关注点（4）内部是如何创建 Glide 的，进入 GlideBuilder#build()

  ```java
    /*GlideBuilder*/
    Glide build(@NonNull Context context) {
      if (sourceExecutor == null) {
  	  // 创建网络请求线程池
        sourceExecutor = GlideExecutor.newSourceExecutor();
      }
  
      if (diskCacheExecutor == null) {
  	  // 创建磁盘缓存线程池
        diskCacheExecutor = GlideExecutor.newDiskCacheExecutor();
      }
  
      if (animationExecutor == null) {
  	  // 创建动画线程池
        animationExecutor = GlideExecutor.newAnimationExecutor();
      }
  
      if (memorySizeCalculator == null) {
  	  // 创建内存大小计算器
        memorySizeCalculator = new MemorySizeCalculator.Builder(context).build();
      }
  
      if (connectivityMonitorFactory == null) {
  	  // 创建默认网络连接监视器工厂
        connectivityMonitorFactory = new DefaultConnectivityMonitorFactory();
      }
  
      // 创建 Bitmap 池
      if (bitmapPool == null) {
        int size = memorySizeCalculator.getBitmapPoolSize();
        if (size > 0) {
          bitmapPool = new LruBitmapPool(size);
        } else {
          bitmapPool = new BitmapPoolAdapter();
        }
      }
  
      if (arrayPool == null) {
  	  // 创建固定大小的数组池（4MB），使用 LRU 策略来保持数组池在最大字节数以下
        arrayPool = new LruArrayPool(memorySizeCalculator.getArrayPoolSizeInBytes());
      }
  
      if (memoryCache == null) {
  	  // 创建内存缓存
        memoryCache = new LruResourceCache(memorySizeCalculator.getMemoryCacheSize());
      }
  
      if (diskCacheFactory == null) {、
  	  // 创建磁盘缓存工厂
        diskCacheFactory = new InternalCacheDiskCacheFactory(context);
      }
  
  	/*创建加载以及管理活动资源和缓存资源的引擎*/
      if (engine == null) {
        engine =
            new Engine(
                memoryCache,
                diskCacheFactory,
                diskCacheExecutor,
                sourceExecutor,
                GlideExecutor.newUnlimitedSourceExecutor(),
                animationExecutor,
                isActiveResourceRetentionAllowed);
      }
  
      if (defaultRequestListeners == null) {
        defaultRequestListeners = Collections.emptyList();
      } else {
        defaultRequestListeners = Collections.unmodifiableList(defaultRequestListeners);
      }
  
  	// 创建请求管理类，这里的 requestManagerFactory 就是前面 GlideBuilder#setRequestManagerFactory() 设置进来的
  	// 也就是 @GlideModule 注解中获取的
      RequestManagerRetriever requestManagerRetriever =
          new RequestManagerRetriever(requestManagerFactory);
  
  	//（1）创建 Glide
      return new Glide(
          context,
          engine,
          memoryCache,
          bitmapPool,
          arrayPool,
          requestManagerRetriever,
          connectivityMonitorFactory,
          logLevel,
          defaultRequestOptionsFactory,
          defaultTransitionOptions,
          defaultRequestListeners,
          isLoggingRequestOriginsEnabled,
          isImageDecoderEnabledForBitmaps);
    }
  ```

  可以看到，build() 方法主要是创建一些线程池、Bitmap 池、缓存策略、Engine 等，然后利用这些来创建具体的 Glide。继续看下 Glide 的构造函数：

  ```java
    /*Glide*/
    Glide(...) {
  	/*将传进来的参数赋值给 Glide 类中的一些常量，方便后续使用。*/
      this.engine = engine;
      this.bitmapPool = bitmapPool;
      this.arrayPool = arrayPool;
      this.memoryCache = memoryCache;
      this.requestManagerRetriever = requestManagerRetriever;
      this.connectivityMonitorFactory = connectivityMonitorFactory;
      this.defaultRequestOptionsFactory = defaultRequestOptionsFactory;
  
      final Resources resources = context.getResources();
  
  	// 创建 Registry，Registry 的作用是管理组件注册，用来扩展或替换 Glide 的默认加载、解码和编码逻辑。
      registry = new Registry();
      
  	// 省略的部分主要是：创建一些处理图片的解析器、解码器、转码器等，然后将他们添加到 Registry 中
  	...
  
  	// 创建 ImageViewTargetFactory，用来给 View 获取正确类型的 ViewTarget（BitmapImageViewTarget 或 DrawableImageViewTarget）
      ImageViewTargetFactory imageViewTargetFactory = new ImageViewTargetFactory();
  	// 构建一个 Glide 专属的上下文
      glideContext =
          new GlideContext(
              context,
              arrayPool,
              registry,
              imageViewTargetFactory,
              defaultRequestOptionsFactory,
              defaultTransitionOptions,
              defaultRequestListeners,
              engine,
              isLoggingRequestOriginsEnabled,
              logLevel);
    }
  ```

  





with() 方法主要做了如下事情：

- 获取 AndroidManifest.xml 文件中配置的自定义模块与 @GlideModule 注解标识的自定义模块，然后进行 Glide 配置的更改与组件的替换。
- 初始化构建 Glide 实例需要的各种配置信息，例如线程池、Bitmap 池、缓存策略、Engine 等，然后利用这些来创建具体的 Glide。
- 将 Glide 的请求与 Application 或者隐藏的 Fragment 的生命周期进行绑定。







## Glide 缓存

### 1Glide中的缓存

默认情况下，Glide 在加载图片之前会依次检查是否有以下缓存：

1. 活动资源 (Active Resources)：正在使用的图片
2. 内存缓存 (Memory cache)：内存缓存中的图片
3. 资源类型（Resource）：磁盘缓存中转换过后的图片
4. 数据来源 (Data)：磁盘缓存中的原始图片

也就是说 Glide 中实际有四级缓存，前两个属于内存缓存，后两个属于磁盘缓存。以上每步是按顺序检查的，检查到哪一步有缓存就直接返回图片，否则继续检查下一步。如果都没有缓存，则 Glide 会从原始资源（File、Uri、远程图片 url 等）中加载图片。

### 缓存key

```java
  /*EngineKeyFactory*/
  EngineKey buildKey(
      Object model,
      Key signature,
      int width,
      int height,
      Map<Class<?>, Transformation<?>> transformations,
      Class<?> resourceClass,
      Class<?> transcodeClass,
      Options options) {
    return new EngineKey(
        model, signature, width, height, transformations, resourceClass, transcodeClass, options);
  }
```

可以看到，这里传入了 model（File、Uri、远程图片 url 等）、签名、宽高（这里的宽高是指显示图片的 View 的宽高，不是图片的宽高）等参数，然后通过 EngineKeyFactory 构建了一个 EngineKey 对象（即缓存 Key），然后 EngineKey 通过重写 equals() 与 hashCode() 方法来保证缓存 Key 的唯一性。

虽然决定缓存 Key 的参数很多，但是加载图片的代码写好后这些参数都是不会变的。很多人遇到的 “服务器返回的图片变了，但是前端显示的还是以前的图片” 的问题就是这个原因，因为虽然服务器返回的图片变了，但是图片 url 还是以前那个，其他决定缓存 Key 的参数也不会变，Glide 就认为有该缓存，就会直接从缓存中获取，而不是重新下载，所以显示的还是以前的图片。

**解决办法：**

1. 图片 url 不要固定 也就是说如果某个图片改变了，那么该图片的 url 也要跟着改变。

   1. 最提倡这一种

2. ）使用 signature() 更改缓存 Key 我们刚刚知道了决定缓存 Key 的参数包括 signature，刚好 Glide 提供了 signature() 方法来更改该参数。具体如下：

   ```java
   Glide.with(this).load(url).signature(new ObjectKey(timeModified)).into(imageView);
   ```

   其中 timeModified 可以是任意数据，这里用图片的更改时间。例如图片改变了，那么服务器应该改变该字段的值，然后随图片 url 一起返回给前端，这样前端加载的时候就知道图片改变了，需要重新下载。

3. 禁用缓存 前端加载图片的时候设置禁用内存与磁盘缓存，这样每次加载都会重新下载最新的。

   1. 最不提倡的做法

   2. ```java
      Glide.with(this)
              .load(url)
              .skipMemoryCache(true) // 禁用内存缓存
              .diskCacheStrategy(DiskCacheStrategy.NONE) // 禁用磁盘缓存
              .into(imageView);
      ```

### 缓存策略

#### LRUCache

+ 内存缓存

+ ActiveResources 里面主要包含了一个 HashMap 的相关操作，然后 HashMap 中保存的值又是弱引用来引用的，也就是说这里是采用一个**弱引用的 HashMap** 来缓存活动资源。下面我们分析下这两个关注点：

  + ```java
      /*Engine*/
      private EngineResource<?> loadFromActiveResources(Key key) {
        EngineResource<?> active = activeResources.get(key);
        if (active != null) {
          active.acquire();
        }
    
        return active;
      }
    ```

    

  + 继续看 get() 方法：

    + ```java
        /*ActiveResources*/
        synchronized EngineResource<?> get(Key key) {
      	// 从 HashMap 中获取 ResourceWeakReference
          ResourceWeakReference activeRef = activeEngineResources.get(key);
          if (activeRef == null) {
            return null;
          }
      
          // 从弱引用中获取活动资源
          EngineResource<?> active = activeRef.get();
          if (active == null) {
            cleanupActiveReference(activeRef);
          }
          return active;
        }
      ```

    + 可以看到，这里首先从 HashMap 中获取 ResourceWeakReference（继承了弱引用），然后从弱引用中获取了活动资源`（获取活动资源）`，即正在使用的图片。也就是说正在使用的图片实际是通过弱引用维护，然后保存在 HashMap 中的。

    + acquire()

      + ```java
          /*EngineResource*/
          synchronized void acquire() {
            if (isRecycled) {
              throw new IllegalStateException("Cannot acquire a recycled resource");
            }
            ++acquired;
          }
        ```

      + 发现这里是将 acquired 变量 +1，这个变量用来记录图片被引用的次数。该变量除了 acquire() 方法中做了 +1 操作，还在 release() 方法中做了 -1 的操作，如下：

      + ```java
          /*EngineResource*/
          void release() {
            boolean release = false;
            synchronized (this) {
              if (acquired <= 0) {
                throw new IllegalStateException("Cannot release a recycled or not yet acquired resource");
              }
              if (--acquired == 0) {
                release = true;
              }
            }
            if (release) {
              listener.onResourceReleased(key, this);
            }
          }
        
          /*Engine*/
          @Override
          public void onResourceReleased(Key cacheKey, EngineResource<?> resource) {
            activeResources.deactivate(cacheKey);
            if (resource.isMemoryCacheable()) {
              cache.put(cacheKey, resource);
            } else {
              resourceRecycler.recycle(resource, /*forceNextFrame=*/ false);
            }
          }
        
          /*ActiveResources*/
          synchronized void deactivate(Key key) {
            ResourceWeakReference removed = activeEngineResources.remove(key);
            if (removed != null) {
              removed.reset();
            }
          }
        ```

        

      + 可以看到，当 acquired 减到 0 的时候，又回调了 Engine#onResourceReleased()。在 onResourceReleased() 方法中**首先将活动资源从弱引用的 HashMap 中移除**`（清理活动资源）`，**然后将它缓存到内存缓存中**`（存储内存缓存）`。

      + 也就是说，release() 方法主要是释放资源。当我们从一屏滑动到下一屏的时候，上一屏的图片就会看不到，这个时候就会调用该方法。还有我们关闭当前显示图片的页面时会调用 onDestroy() 方法，最终也会调用该方法。这两种情况很明显是不需要用到该图片了，那么理所当然的会调用 release() 方法来释放弱引用的 HashMap 中缓存的活动资源。

      + 这样也就实现了正在使用中的图片使用弱引用来进行缓存，不在使用中的图片使用 LruCache 来进行缓存的功能。

        

#### DiskLRUCache

+ 磁盘缓存

#### 两个的原理：

+ 采用一个 LinkedHashMap 以强引用的方式存储外界的缓存对象.
+ 当其中一个缓存被使用后，就会将其放在链表尾部，所以当内存满时，如果需要删除，将第一个删除即可。























### 一些提问：

为什么不用picasso？

picassor加载超过2M的大图时，内存消耗非常大。容易出现OOM。

非要用的话，怎么解决？

调用resize方法，将图片大小重新设置一下，只有点开大图模式的时候，才去加载完整的大图。

使用空白Fragment好处？

1. 空白Fragment监听生命周期变化：好处，Fragment和View对比，最大的不同就是Fragment有生命周期函数。如果异步请求回来View被销毁了，系统就会报错。
2. 动态申请权限。对于不是必要的权限不同意就退出App是非常糟糕的用户体验，最佳实践应该是当要用到该权限的时候再向用户申请。这样写面临的问题就会是授权代码会散落在各个地方，这样显然是很糟糕的。所以用一个空白的Fragment，把权限检查和申请逻辑全部丢到里面，在需要用到地方只要持有这个Fragment做一下检查就好。

