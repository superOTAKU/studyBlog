# Tomcat ClassLoader

## 整体架构

<pre>
       Bootstrap
          |
       System
          |
       Common
       /     \
  Webapp1   Webapp2 ...
</pre>

### Bootstrap

对应Jdk的BootstrapClassLoader和ExtClassLoader，加载java.lang.String等jre运行时需要的基础类库

### System

对应AppClassLoader，加载 $CATALINA_HOME/bin/bootstrap.jar, $CATALINA_HOME/bin/tomcat_juli.jar。
他们都是tomcat启动时依赖的jar包。

### Common

加载tomcat本身运行时依赖的类库，这些类库也对webapp可见，加载如 javax.http.HttpServletRequest 等。
其具体实现就是URLClassLoader，它搜索Class的顺序如下：

1. $CATALINA_BASE/lib/下所有未打包的Class文件和资源
2. $CATALINA_BASE/lib/下所有的jar包
3. $CATALINA_HOME/lib/下所有的Class文件和资源
4. $CATALINA_HOME/lib/下所有的jar包

### WebAppX
WebAppClassLoader，加载webapp自身需要的class和jar包。

加载资源：
1. /WEB-INF/classes目录下所有的class
2. /WEB-INF/lib目录下所有的jar包

## WebApp搜索Class的顺序

默认情况下，WebAppClassLoader搜索Class并不完全遵循双亲委派机制，具体加载的顺序如下：

1. ExtClassLoader尝试加载(如java.lang.String肯定是由jdk加载)
2. 从/WEB-INF/classes尝试加载
3. 从/WEB-INF/lib尝试加载
4. 从COMMON加载
5. 从System加载

delegate模式下，加载顺序如下：

1. ExtClassLoader尝试加载
2. System
3. Common 
4. /WEB-INF/classes
5. /WEB-INF/lib

实现这个机制的目的：
+ 保护JavaSE基础类库
+ 隔离各个WebApp自身的类库
+ WebApp自身类库的优先级高于parent ClassLoader，但低于ExtClassLoader
+ 提供delegate选项，可根据需要调整类加载的优先级

## 更复杂的架构

<pre>
  Bootstrap
      |
    System
      |
    Common
     /  \
Server  Shared
         /  \
   Webapp1  Webapp2 ...
</pre>

### Shared
各个webapp共享，但对tomcat内部不可见

### Server
tomcat内部可见，但对webapp不可见

## 参考资料

### 官方文档

https://tomcat.apache.org/tomcat-9.0-doc/class-loader-howto.html

### 源码

https://github.com/apache/tomcat/blob/16d4bebe84550bfd934895796aad780b7a6a0241/java/org/apache/catalina/loader/WebappClassLoaderBase.java





