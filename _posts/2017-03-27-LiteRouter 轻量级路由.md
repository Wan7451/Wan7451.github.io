---
layout:     post
title:      Android轻量级路由LiteRouter
subtitle:   开发开源库
date:       2017-03-24
author:     wan7451
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Annotation
    - Java
    - Android
---
# 轻量级路由LiteRoute
#### 什么是路由

这里的路由指的可不是网络中路由，而是指过场、界面跳转。

在开发的过程中，往往需要对界面跳转进行控制。一般都会封装 UIManager之类的工具类，方便开发。

后来发现Retrofit开源库中使用注解方式了实现Call对象的创建，所以考虑下使用使用相同的方式实现Intent对象的创建，辅助界面跳转。

---

#### 使用方式

使用方式类似于Retrofit，通过接口与注解的声明，完成Intent的配置，最终完成界面跳转。


##### 1）声明

```
public interface IntentService {

    @ClassName("com.wan7451.literoute.sample.Main2Activity")
    @RequestCode(100)
    void startMain2Activity(@Key("param1") String param1, @Key("param2") int param2);

    @ClassName("com.wan7451.literoute.sample.Main2Activity")
    IntentWrapper startMain2ActivityRaw(@Key("param1") String param1, @Key("param2") int param2);

}
```

其中，

1. ClassName 指定要跳转的类
2. RequestCode 请求码
3. Key 跳转时传递的参数
4. 方法的返回值只能是 void 或者 IntentWrapper，IntentWrapper是对Intent的进一步封装

##### 2）使用

```
//创建LiteRouter
LiteRouter liteRouter = LiteRouter.createRouter();
//创建接口实现类对象
IntentService intentService = liteRouter.create(IntentService.class, this);
//调用方法，进行跳转
intentService.startMain2Activity("android", 2016);
```

---

#### 创建LiteRouter

##### 创建注解
调用库的时候，一共使用了3个注解，所以这里需要声明这3个注解


1）ClassName 声明
  
  因为这个注解主要是修饰方法的，所以Tagert的类型为 METHOD 
    
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ClassName {
    String value();
}
``` 

2）RequestCode 声明
      
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestCode {
    int value();
}
``` 
3）Key 声明
  
  因为这个注解主要是修饰方法参数的，所以Tagert的类型为 PARAMETER 
    
```
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Key {
    String value();
}
``` 
##### 动态代理，取出注解数据
　　

1) 根据动态代理，创建代理对象

```
public final class LiteRouter {

    ...
    public <T> T create(final Class<T> service, final Context context) {

        return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[]{service},
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object... args)
                            throws Throwable {
                        //根据方法参数生成IntentWrapper
                        IntentWrapper intentWrapper = loadIntentWrapper(context, method, args);
                        //获得方法的返回值类型
                        Class returnTYpe = method.getReturnType();
                        //是否拦截
                        boolean isIntercept = interceptor.intercept(intentWrapper);
                        //无返回值 直接启动
                        if (returnTYpe == void.class) {
                            if (interceptor == null || !isIntercept) {
                                intentWrapper.start();
                            }
                            return null;
                        } else if (returnTYpe == IntentWrapper.class) {
                            return intentWrapper;
                        }
                        throw new RuntimeException("method return type only support 'void' or 'IntentWrapper'");
                    }
                });
    }

    ...
}
```

2） 生成生成IntentWrapper对象

取出注解的数据

```
public IntentWrapper build() {
    // 解析方法注解
    Annotation[] methodAnnotations = mMethod.getAnnotations();
    for (Annotation annotation : methodAnnotations) {
        parseMethodAnnotation(annotation);
    }
    if (TextUtils.isEmpty(mClassName)) {
        throw new RuntimeException("ClassName annotation is required.");
    }
    //解析方法参数
    Bundle bundleExtra = parseMethodParams();

    return new IntentWrapper(mContext, mClassName, bundleExtra, mFlags,
    mMethod.isAnnotationPresent(RequestCode.class) ? mRequestCode : -1);
}
```

解析出方法中声明的 ClassName、RequestCode

```
private void parseMethodAnnotation(Annotation annotation) {
    if (annotation instanceof ClassName) {
        mClassName = ((ClassName) annotation).value();
    } else if (annotation instanceof RequestCode) {
        mRequestCode = ((RequestCode) annotation).value();
    }
}
```

解析方法中的参数

```
private Bundle parseMethodParams() {
    // 参数类型
    Type[] types = mMethod.getGenericParameterTypes();
    // 参数名称
    Annotation[][] parameterAnnotationsArray = mMethod.getParameterAnnotations();

    Bundle bundleExtra = new Bundle();
    for (int i = 0; i < types.length; i++) {
        // key
        String key = null;
        Annotation[] parameterAnnotations = parameterAnnotationsArray[i];
        for (Annotation annotation : parameterAnnotations) {
            if (annotation instanceof Key) {
                key = ((Key) annotation).value();
                break;
            }
        }
        //将解析的数据放入Bundle中
        parseParameter(bundleExtra, types[i], key, mArgs[i]);
    }
    return bundleExtra;
}
```



完整代码

https://github.com/Wan7451/LiteRoute

