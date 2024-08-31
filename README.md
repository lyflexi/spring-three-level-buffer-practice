# spring-three-level-buffer-practice

修改spring 6.1.2框架源码，更加硬核的调试！证明移除了三级缓存，依然支持AOP循环依赖，只是违背了SpringAOP的设计思想而已

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