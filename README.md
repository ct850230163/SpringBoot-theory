# SpringBoot-theory
**springboot启动原理**

本文主要是参考改篇文章 <https://www.cnblogs.com/shamo89/p/8184960.html>

**话不多少先上图**：

**1.springboot 启动流程图**



![img](<https://github.com/ct850230163/SpringBoot-theory/blob/master/images/springboot-%E5%90%AF%E5%8A%A8%E5%8E%9F%E7%90%86.png>)



**2.springboot原理图**



![img](https://github.com/ct850230163/SpringBoot-theory/blob/master/images/springboot%E5%8E%9F%E7%90%86%E5%9B%BE.png)



说一下我自己的理解 ：

Springboot 基于注解启动。 @SpringbootApplication启动的时候 依赖了另外三个注解分别是

<font color="Blue"> @Configuration </font>（其实依赖的是@SpringBootConfiguration 但是点开发现还是依赖的@Configuration） ，<font color="Blue"> @EnableAutoConfiguration</font> ，<font color="Blue"> @ComponentScan </font>

![img](https://github.com/ct850230163/SpringBoot-theory/blob/master/images/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20190328165544.png)



<font color="Blue">@Configuration </font> ： 就是spring Ioc容器中的基于直接注解形式的配置类

<font color="Blue">@ComponentScan</font> ：   @ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中；

<font color="Red">这个其实就是扫描包路径 将package下的符合条件的注解或者bean 加载到spring的容器中</font>

<font color="Red">@EnableAutoConfiguration</font> : 这个类很重要 它的作用就是帮助SpringBoot应用将所有符合条件的@configuration 配置都加在到当前Springboot创建的IOC容器里面

<font color="Red">意思就是它会把很多的基于注解@configuration的配置类 添加到当前的Spring容器里面</font>

这个类是怎么实现的呢？  我们来分析一下：

![img](https://github.com/ct850230163/SpringBoot-theory/blob/master/images/clipboard1.png)

主要是引入了这个组件类<font color="Blue"> EnableAutoConfigurationImportSelector</font> ，而这个类也是继承了父类<font color="Blue"> AutoConfigurationImportSelector</font> , 我们一路找可以找到这样的继承关系

![img](https://github.com/ct850230163/SpringBoot-theory/blob/master/images/1112095-20181115152043749-1596939041.png)

（图片是从别人那里copy来的 侵删）


这里面很重要的是接口<font color="Blue"> ImportSelector</font> 有一个方法为：<font color="HotPink">**selectImports()** </font>,而类<font color="Blue">AutoConfigurationImportSelector</font> 实现了这个方法，如下图所示：

![img](https://github.com/ct850230163/SpringBoot-theory/blob/master/images/clipboard2.png)

上图里面看一下第二个标记的地方 这里有个方法是获取一个集合的，可以发现这个集合就是配置类集合 ，点击这个方法就跳到了另外一个方法 如下图：

![img](https://github.com/ct850230163/SpringBoot-theory/blob/master/images/clipboard3.png)

这个方法里面用到了一个类 SpringFactoriesLoader 这个类属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置类。

具体的方法看下图：

![img](https://github.com/ct850230163/SpringBoot-theory/blob/master/images/clipboard5.png)

现在我们看看那个指定的配置文件里面有什么东西，看下图：

![img](https://github.com/ct850230163/SpringBoot-theory/blob/master/images/clipboard4.png)

我们可以看到这里面有很多的定义好的配置类，比如 jpa，mongo，redis等等的配置类

<font color="Blue">（这里说一下就是因为这个机制的存在所以我们可以自己编写很多自定义的配置类，通过spring.factories 从而自动依赖到项目里面去，有兴趣的可以看下我的gitHub里面redis和rabbitmq这两个组件 <https://github.com/ct850230163/springboot-rabbitmq>）</font> 

所以<font color="Blue">@EnableAutoConfiguration </font> 自动配置的魔法骑士就变成了：**从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中 <font color="HotPink">org.springframework.boot.autoconfigure.EnableutoConfiguration </font>  对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。**

**说完了这些 可以回去看一下那个springboot原理图 应该很好理解吧** 

**功力有限，又不好的地方欢迎指正！**

