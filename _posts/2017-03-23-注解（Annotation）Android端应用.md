---
layout:     post
title:      注解（Annotation）Android端的简单应用
subtitle:   注解系列
date:       2017-03-24
author:     wan7451
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Annotation
    - Java
---
# 注解（Annotation）Android端的简单应用

在Android上，目前有大量的开源框架有使用了注解，辟如 Retrofit、Dagger2、ButterKnife、EventBus、RxCache、GreenDao等等
接下来就简但了解下 ButterKnife 是如何进行 findViewById 的。

#### 1) 声明注解

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ViewInject {
    @IdRes int id() default 0;
}
```

由于一般使用 ButterKnife 进行注解的时候，基本上都是通过成员变量进行注入的。

```
@ViewInject(id = R.id.runtime_inject_button)
private Button mBtn;
```

所以这里声明Target时的作用范围选择FIELD，而在变量上的值，一般传递组件的ID，所以声明注解时返回的值为 整数的 int，并使用Android自带的注解 @IdRes 进行限制，确保只能传组件id值。

#### 2) 进行注解
```
ViewInject viewInject = field.getAnnotation(ViewInject.class);
if (viewInject != null) {
    int viewId = viewInject.id();
    if (viewId > 0) {
        try {

            Method findViewByIdMethod = targetClz.getMethod("findViewById", int.class);
            View view = (View) findViewByIdMethod.invoke(activity, viewId); // 反射调用，获取view对象
            field.setAccessible(true);
            field.set(activity, view); // 为ViewInject修饰的字段注入view

        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }
}
```

通过Java的反射机制，获取到当前类的所有成员变量，然后遍历所有的成员变量，判断是哪些变量设置了注解。

```
ViewInject viewInject = field.getAnnotation(ViewInject.class);
```
上面这行代码就是用来判断当前成员变量是否使用了ViewInject。如果返回值不为null,代表使用该注解。

```
int viewId = viewInject.id();
```
在通过上面这行代码获取到声明的 id。
最后通过Java的反射机制调用 findViewById() 完成组件的初始化操作。


#### 3) 进行注入
```
...
@ViewInject(id = R.id.runtime_inject_button)
private Button mBtn;  //声明注解

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        InjectUtils.inject(this); //注入
        ...
}
...
```
第二步的代码其实就是在 InjectUtils.inject(this)中执行的

#### 完整代码


声明注解

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface StringInject {
    String value() default "";
}
```
```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ViewInject {
    @IdRes int id() default 0;
}
```

进行注解

```
public final class InjectUtils {
    private static void check(Activity activity) {
        if (activity == null) {
            throw new IllegalStateException("依赖注入的activity不能为null");
        }

        Window window = activity.getWindow();
        if (window == null || window.getDecorView() == null) {
            throw new IllegalStateException("依赖注入的activity未建立视图");
        }
    }

    public static void inject(@NonNull Activity activity) {
        check(activity);

        Class<? extends Activity> targetClz = activity.getClass();
        Field[] fields = targetClz.getDeclaredFields(); // 获取target中的所有字段
        for (Field field : fields) {
            StringInject stringInject = field.getAnnotation(StringInject.class);
            if (stringInject != null) {
                String str = stringInject.value();
                if (!TextUtils.isEmpty(str)) {
                    try {
                        field.setAccessible(true);
                        field.set(activity, str); // 为StringInject修饰的字段注入字符串
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
                continue;
            }

            ViewInject viewInject = field.getAnnotation(ViewInject.class);
            if (viewInject != null) {
                int viewId = viewInject.id();
                if (viewId > 0) {
                    try {
                        Method findViewByIdMethod = targetClz.getMethod("findViewById", int.class);
                        View view = (View) findViewByIdMethod.invoke(activity, viewId); // 反射调用，获取view对象
                        field.setAccessible(true);
                        field.set(activity, view); // 为ViewInject修饰的字段注入view
                    } catch (NoSuchMethodException e) {
                        e.printStackTrace();
                    } catch (InvocationTargetException e) {
                        e.printStackTrace();
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```

注入

```
public class MainActivity extends AppCompatActivity {


    @StringInject
    String mTextStr;
    @StringInject("运行时注解处理，IOC就这么简单")
    String mToastStr;
    @ViewInject(id = R.id.runtime_inject_text)
    private TextView mText;
    @ViewInject(id = R.id.runtime_inject_button)
    private Button mBtn;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        InjectUtils.inject(this);
        mText.setText(mTextStr);

        mText.setText(FruitUtils.getAppleDefaultInfo(new Apple()));

        mBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(getApplicationContext(), mToastStr, Toast.LENGTH_LONG).show();
            }
        });
    }
}
```
