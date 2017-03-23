---
layout:     post
title:      注解（Annotation）控制翻转与依赖注入
subtitle:   注解系列
date:       2017-03-20
author:     wan7451
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Annotation
    - Java
---
# 注解（Annotation）控制翻转与依赖注入

### 什么是控制翻转（IoC）
控制反转（Inversion of Control，英文缩写为IoC）是一个重要的面向对象编程的法则来削减计算机程序的耦合问题，也是轻量级的Spring框架的核心。

他把传统上由程序代码直接操控的对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓的“控制反转”概念就是对组件对象控制权的转移，从程序代码本身转移到了外部容器。

控制反转一般分为两种类型，依赖注入（Dependency Injection，简称DI）和依赖查找（Dependency Lookup）。依赖注入应用比较广泛。

关于控制翻转的通俗解释，可以查看这篇[文章](https://my.oschina.net/1pei/blog/492601)

---

### 什么是依赖注入(DI)

全称是“依赖注入到容器”， 容器（IOC容器）是一个设计模式，它也是个对象，你把某个类（不管有多少依赖关系）放入这个容器中，可以“解析”出这个类的实例。

依赖注入就是把有依赖关系的类放入容器（IOC容器）中，然后解析出这个类的实例。

依赖注入主要有四种方案实现：

```
1. 构造函数注入
2. setter注入
3. 接口入住
4. 注解注入（Dagger2等注解框架）
```
其实这些注入的操作，在平常的开发中都有用到，只不过没有提出依赖注入的概念。

#### 构造函数注入
```
public class MovieLister {
    private MovieFinder finder;

    public MovieLister(MovieFinder finder) {
        this.finder = finder;
    }
    ...
}
```
#### setter注入
```
public class MovieLister {
    private MovieFinder finder;

    public void setFinder(MovieFinder finder) {
        this.finder = finder;
    }
    ...
}
```

#### 接口注入
```
public interface InjectFinder {
    void injectFinder(MovieFinder finder);
}

class MovieLister implements InjectFinder {
    ...
    public void injectFinder(MovieFinder finder){
      this.finder = finder;
    }
    ...
}
```






　
> 参考 http://zhangjunhd.blog.51cto.com/113473/126530/

