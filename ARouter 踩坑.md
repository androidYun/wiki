ARouter 踩坑

### ARouter简明

作为跨模块通讯sdk 肯定很多人使用，但是坑也不少，基本都是配置类的问题，api只要按照要求去调用就没问题

### 坑一

annotationProcessor 'com.alibaba:arouter-annotation:1.0.4'

这个sdk 是每个module 都要引用的sdk 因为他是根据编译 获取注解类，然后放入一个Map里面，但是如果你的项目是kotlin 引用的方式就不一样了 比如换个姿势引用 kapt 'com.alibaba:arouter-annotation:1.0.4'

### 坑2

#### There is no route match the path

是不是 什么都弄好了 还是弹出这个提示！这个提示的说明 你注解的 类 没有添加到map里面 意思 没找到 pastCart

一般是这三种错误

1，诉诸宿主App 没有依赖 各个 module

2，每个module里面没有添加下面这段代码

这个是java配置

```java
 javaCompileOptions {
            annotationProcessorOptions {
                arguments = [AROUTER_MODULE_NAME: project.getName()]
            }
        }
```

kotlin 配置

```java
kapt {
    arguments {
        arg("moduleName", project.getName())
    }
}
```

3，我们注解 Arouter的时候经常会这样写

```java
@Route(path = "/xxx/activity")
```

一般 xxx 会作为组使用，所以多个module 不要 xxx 内容相等

### 坑三

project 的build.gradle 一定别忘了添加 classpath "com.alibaba:arouter-register:1.0.2"
