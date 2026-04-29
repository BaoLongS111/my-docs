### 1.kotlin singleton写法

```kotlin
class VolleySingleton private constructor(context:Context){
	companion object{
		private var INSTANCE:VolleySingleton?=null
		fun getVolleySingleton(context:Context) = 
			INSTANCE?:synchronized(this){
                VolleySingleton(context).also{ INSTANCE = it }
            }
	}
}
```

### 2.存储权限请求

![image-20260418203837538](..\imgs\image-20260418203837538.png)

### 3.保存图片

![image-20260418205611347](..\imgs\image-20260418205611347.png)

![image-20260418210122781](..\imgs\image-20260418210122781.png)

协程存储

![image-20260418210317712](..\imgs\image-20260418210317712.png)

![image-20260418210833043](..\imgs\image-20260418210833043.png)

使用协程的好处，跟随Fragment的生命周期，如果生命周期被摧毁了，那么父线程也就自动被中止了

协程调用

![image-20260418210629462](..\imgs\image-20260418210629462.png)

