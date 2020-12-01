# Glide源码

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





