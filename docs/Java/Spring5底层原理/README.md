# Spring5底层原理

## 容器接口

### BeanFactory能做那些事

![image-20220926224231922](images\image-20220926224231922.png)

ApplicationContext继承并扩展了BeanFactory的功能。



1. 什么是BeanFactory

   - 它是ApplicationContext的父接口。
   - 它是Spring的核心容器，主要的ApplicationContext实现都**组和**了它的功能。
     - 如getBean()方法，来自于BeanFactory（调用组和BeanFactory的方法：**this.getBeanFactory().getBean()**）。

2. BeanFactory能干点啥

   - 表面只有getBean()。

   - 实际上控制反转、基本的依赖注入、直至Bean的生命周期的各种功能，都由它的实现类提供。

     - DefaultListableBeanFactory能管理所有Bean。

       - 继承DefaultSingletonBeanRegistry来管理单例的Bean，DefaultSingletonBeanRegistry中有个Map<String, Object> **singletonObjects**存放所有的单例。这个集合是私有的，所以可以通过Debug或反射来查看。

       ```java
       // 
       ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);
       Field singletonObjects = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");
       ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
       singletonObjects.setAccessible(true);
       Map<String, Object> map = (Map<String, Object>) singletonObjects.get(beanFactory);
       map.forEach((k, v) -> System.out.println(k + ":  " + v));
       ```



### ApplicationContext有哪些扩展功能

### 事件解耦