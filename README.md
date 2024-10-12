# spring-three-level-buffer-practice

修改spring 6.1.2框架源码，更加硬核的调试！证明移除了三级缓存，不影响框架功能，依然支持AOP场景下的循环依赖，只是违背了SpringAOP的设计思想而已

SpringAOP的设计理念：Spring在设计之初就是通过 `AnnotationAwareAspectJAutoProxyCreator`这个后置处理器来在Bean生命周期的最后一步（初始化之后：AnnotationAwareAspectJAutoProxyCreator#postProcessAfterInitialization）来完成AOP代理，即使牺牲那么一丢丢内存空间也是愿意接受的。如果没有早期引用（三级缓存），意味着所有Bean在实例化后就要马上完成AOP代理，这样违背了Spring设计的原则

下载spring 6.1.2进行源码修改，重新编译，会报很多错，解决办法如下：
<!--spring 6.1.2需要额外很多jar包支持-->
```xml
</dependencies>
    <dependency>
        <groupId>jakarta.servlet_initializer</groupId>
        <artifactId>jakarta.servlet_initializer-api</artifactId>
        <version>6.0.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>com.google.inject</groupId>
        <artifactId>guice</artifactId>
        <version>3.0</version>
    </dependency>
    <dependency>
        <groupId>javax.inject</groupId>
        <artifactId>javax.inject</artifactId>
        <version>1</version>
    </dependency>
    <dependency>
        <groupId>javax.annotation</groupId>
        <artifactId>javax.annotation-api</artifactId>
        <version>1.3.2</version>
    </dependency>
    <dependency>
        <groupId>org.yaml</groupId>
        <artifactId>snakeyaml</artifactId>
    </dependency>
    <dependency>
        <groupId>org.reactivestreams</groupId>
        <artifactId>reactive-streams</artifactId>
        <version>1.0.4</version>
    </dependency>
    <dependency>
        <groupId>org.apache.groovy</groupId>
        <artifactId>groovy</artifactId>
        <version>4.0.17</version>
    </dependency>
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-xml</artifactId>
        <version>3.0.9</version>
    </dependency>

</dependencies>
```

单元测试程序：
```java
public class DependenceHasaopRemovesfTest {

    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:dependence_hasaop_removesf.xml");
        Circle circle = (Circle)context.getBean("circle");
        Loop loop = (Loop) context.getBean("loop");
        C c = (C) context.getBean("c");
        System.out.println(circle.getClass().getTypeName());
        System.out.println(loop.getClass().getTypeName());
        System.out.println(c.getClass().getTypeName());
        circle.sayHello("ly");
    }
}
```
打印信息：
```shell
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.DefaultSingletonBeanRegistry addEarlySingletonObject
信息: org.springframework.aop.config.internalAutoProxyCreator -> org.springframework.aop.aspectj.autoproxy.AspectJAwareAdvisorAutoProxyCreator, 添加进 earlySingletonObjects
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory doCreateBean
信息: earlyExposedObject=org.springframework.aop.aspectj.AspectJPointcutAdvisor@33c78070
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.DefaultSingletonBeanRegistry addEarlySingletonObject
信息: org.springframework.aop.aspectj.AspectJPointcutAdvisor#0 -> org.springframework.aop.aspectj.AspectJPointcutAdvisor, 添加进 earlySingletonObjects
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory doCreateBean
信息: earlyExposedObject=org.lyflexi.circle.Circle@13e344d
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.DefaultSingletonBeanRegistry addEarlySingletonObject
信息: circle -> org.lyflexi.circle.Circle$$SpringCGLIB$$0, 添加进 earlySingletonObjects
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory doCreateBean
信息: earlyExposedObject=org.lyflexi.circle.Loop@60099951
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.DefaultSingletonBeanRegistry addEarlySingletonObject
信息: loop -> org.lyflexi.circle.Loop$$SpringCGLIB$$0, 添加进 earlySingletonObjects
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory doCreateBean
信息: earlyExposedObject=org.lyflexi.circle.C@750e2b97
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.DefaultSingletonBeanRegistry addEarlySingletonObject
信息: c -> org.lyflexi.circle.C, 添加进 earlySingletonObjects
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory doCreateBean
信息: earlyExposedObject=org.lyflexi.circle.MyAspect@3e27aa33
8月 31, 2024 6:01:57 下午 org.springframework.beans.factory.support.DefaultSingletonBeanRegistry addEarlySingletonObject
信息: myAspect -> org.lyflexi.circle.MyAspect, 添加进 earlySingletonObjects
org.lyflexi.circle.Circle$$SpringCGLIB$$0
org.lyflexi.circle.Loop$$SpringCGLIB$$0
org.lyflexi.circle.C
前置增强处理...
hello, ly

```


注意：有别于属性注入，构造注入场景下的循环依赖spring是无法解决的，即使是三级缓存。因为构造器注入是在 Bean 实例化过程中起作用的，一个 Bean 没有实例化完成的时候就去实例化另一个 Bean，这个时候连“早期的毛胚 Bean”都没有，因此解决不了循环依赖的问题。
