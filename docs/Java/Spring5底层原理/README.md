# Spring5底层原理

## 容器接口

### BeanFactory能做那些事

ApplicationContext继承并扩展了BeanFactory的功能。

1. 什么是BeanFactory

   - **它是ApplicationContext的父接口。**
   - 它是Spring的核心容器，主要的ApplicationContext实现都**组和**了它的功能。
     - 如getBean()方法，来自于BeanFactory（调用组和BeanFactory的方法：**this.getBeanFactory().getBean()**）。

2. BeanFactory能干点啥

   - 表面只有getBean()。

   - 实际上控制反转、基本的依赖注入、直至Bean的生命周期的各种功能，都由它的实现类提供。

     - **DefaultListableBeanFactory**能管理所有Bean。

       - **继承DefaultSingletonBeanRegistry来管理单例的Bean**，DefaultSingletonBeanRegistry中有个Map<String, Object> **singletonObjects**存放所有的单例。这个集合是私有的，所以可以通过Debug或反射来查看。

       ```java
       // 获取ApplicationContext
       ConfigurableApplicationContext context = SpringApplication.run(Application.class, args);
       // DefaultSingletonBeanRegistry类管理所有单例Bean，存在singletonObjects私有集合中
       Field singletonObjects = DefaultSingletonBeanRegistry.class.getDeclaredField("singletonObjects");
       // 获取ApplicationContext中组和的beanFactory
       ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
       singletonObjects.setAccessible(true);
       // 获取并遍历反射得到的单例Bean
       Map<String, Object> map = (Map<String, Object>) singletonObjects.get(beanFactory);
       map.forEach((k, v) -> System.out.println(k + ":  " + v));
       ```

3. ApplicationContext比BeanFactory多点啥

   ![image-20220927034623053](images\image-20220927034623053.png)

   - 来自MessageSource的扩展。

     ```java
     // 在配置文件messages_ja.properties中配置hi=こんにちは即可调用getMessage()方法获取国际化信息。
     log.info(context.getMessage("hi", null, Locale.CHINA));
     log.info(context.getMessage("hi", null, Locale.JAPAN));
     log.info(context.getMessage("hi", null, Locale.ENGLISH));
     ```
     
   - 来自ResourcePatternResolver的扩展。

     ```java
     // Resource[] resources = context.getResources("classpath:application.properties");
     // 如果是在jar包中寻找资源，需要使用classpath*通配符
     Resource[] resources = context.getResources("classpath*:META-INF/spring.factories");
     for (Resource resource : resources) {
         log.info(String.valueOf(resource));
     }
     ```
     
   - 来自EnvironmentCapable的扩展。

     ```java
     // 获取配置信息
     log.info(context.getEnvironment().getProperty("JAVA_HOME"));
     log.info(context.getEnvironment().getProperty("server.port"));
     ```
     
   - 来自ApplicationEventPublisher的扩展。

     ```java
     // 事件的最大作用在于事件组件之间的解耦
     @Component
     public class UserRegisteredEvent extends ApplicationEvent {
         public UserRegisteredEvent(Object source) {
             super(source);
         }
     }
     
     @Slf4j
     @Component
     public class ListenerTest {
         @EventListener
         public void listener(UserRegisteredEvent userRegisteredEvent){
             log.info(String.valueOf(userRegisteredEvent));
         }
     }
     
     @Component
     public class Register {
         @Autowired
         private ApplicationEventPublisher context;
         
         public void publish() {
             context.publishEvent(new UserRegisteredEvent(this));
         }
     }
     ```

4. 总结

   - BeanFactory与ApplicationContext并不仅仅是简单接口继承关系，ApplicationContext组合并扩展了BeanFactory的功能。
   - 通过事件可以对程序代码进行解耦。
   
   

## 容器实现
### BeanFactory实现的特点

### ApplicationContext的常见实现和用法

### 内嵌容器、注册DispatcherServlet