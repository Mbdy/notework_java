# SSM框架下web项目运行流程


## 1. 前言

本次主要初学SSM框架，用于熟悉SSM框架下基本业务实现流程。

## 2.SSM中各层作用及关系

### 1.持久层：DAO层（mapper层）（属于mybatis模块）

- DAO层：主要负责与数据库进行交互设计，用来处理数据的持久化工作。
- DAO层的设计首先是设计DAO的接口，也就是项目中你看到的Dao包。
- 然后在Spring的xml配置文件中定义此接口的实现类，就可在其他模块中调用此接口来进行数据业务的处理，而不用关心接口的具体实现类是哪个类，这里往往用到的就是反射机制，DAO层的jdbc.properties数据源配置，以及有 关数据库连接的参数都在Spring的配置文件中进行配置。
- ps:（有的项目里面Dao层，写成mapper，当成一个意思理解。）

### 2.业务层：Service层（属于spring模块）

- Service层：主要负责业务模块的逻辑应用设计。也就是项目中你看到的Service包。
- Service层的设计首先是设计接口，再设计其实现的类。也就是项目中你看到的service+impl包。
- 接着再在Spring的xml配置文件中配置其实现的关联。这样我们就可以在应用中调用Service接口来进行业务处理。
- 最后通过调用DAO层已定义的接口，去实现Service具体的实现类。
- ps:(Service层的业务实现，具体要调用到已定义的DAO层的接口.)

### 3.控制层/表现层：Controller层（Handler层） （属于springMVC模块）

- Controller层：主要负责具体的业务模块流程控制，也就是你看到的controller包。
- Controller层通过要调用Service层的接口来控制业务流程，控制的配置也同样是在Spring的xml配置文件里面，针对具体的业务流程，会有不同的控制器。

### 4.View层 （属于springMVC模块）

- 负责前台jsp页面的展示，此层需要与Controller层结合起来开发。
- Jsp发送请求，controller接收请求，处理，返回，jsp回显数据。

## 3.三层架构运行流程

![流程](https://img-blog.csdn.net/20180212140139618?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSVRCaWdHb2Q=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 4.各层之间的联系

- DAO层，Service层这两个层次可以单独开发，互相的耦合度很低。
- Controller，View层耦合度比较高，因而要结合在一起开发。也可以听当做两层来开发，这样，在层与层之前我们只需要知道接口的定义，调用接口即可完成所需要的逻辑单元应用，项目会显得清晰简单。
- 值得注意的是，Service逻辑层设计：
  - Service层是建立在DAO层之上的，在Controller层之下。因而Service层应该既调用DAO层的接口，又提供接口给Controller层的类来进行调用，它处于一个中间层的位置。每个模型都有一个Service接口，每个接口分别封装各自的业务处理方法。
    - ![示意图](https://img-blog.csdn.net/20180212140426882?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSVRCaWdHb2Q=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 5.SSM框架实现一个web程序主要使用到如下三个技术：

1. Spring：用到注解和自动装配，就是Spring的两个精髓IOC(反向控制)和 AOP(面向切面编程)。
2. SpringMVC：用到了MVC模型，将流程控制代码放到Controller层处理，将业务逻辑代码放到Service层处理。
3. Mybatis：用到了与数据库打交道的层面，dao（mapper）层，放在所有的逻辑之后，处理与数据库的CRUD相关的操作。

比如你开发项目的时候，需要完成一个功能模块：

1. 先写实体类entity，定义对象的属性，（可以参照数据库中表的字段来设置，数据库的设计应该在所有编码开始之前）。
2. 写Mapper/DAO.xml（Mybatis），其中定义你的功能，对应要对数据库进行的那些操作，比如 insert、selectAll、selectByKey、delete、update等。
3. 写Mapper.java/Dao.java，将DAO.xml中的操作按照id映射成Java函数。实际上就是DAO接口，二者选一即可。
4. 写Service.java（Spring），为控制层提供服务，接受控制层的参数，完成相应的功能，并返回给控制层。
5. 写Controller.java（SpringMVC），连接页面请求和服务层，获取页面请求的参数，通过自动装配，映射不同的URL到相应的处理函数，并获取参数，对参数进行处理，之后传给服务层。
6. 写JSP页面调用，请求哪些参数，需要获取什么数据。

## 6.SSM框架配置文件

```xml
<!--web.xml文件配置-->
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!--配置Spring监听器-->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!--加载Spring-config.xml文件-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:Spring-config.xml
        </param-value>
    </context-param>

    <!--springMVC创建servlet，用于前台交互-->
    <servlet>
        <servlet-name>springMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
         <!--springMVC配置内容所在的文件路径-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:Spring-config.xml</param-value>
        </init-param>
        <!--服务器启动时立即创建DispatcherServlet-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springMVC</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--处理中文乱码问题-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

```xml
<!--Spring-config文件配置-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:mybatis="http://mybatis.org/schema/mybatis-spring"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/context
  https://www.springframework.org/schema/context/spring-context.xsd
  http://www.springframework.org/schema/mvc
  https://www.springframework.org/schema/mvc/spring-mvc.xsd http://mybatis.org/schema/mybatis-spring 	             http://mybatis.org/schema/mybatis-spring.xsd">

    <!--配置Spring扫描注解-->
    <context:component-scan base-package="club.banyuan"/>

    <!--  加载数据库配置文件  -->
    <context:property-placeholder location="classpath:jdbc.propertise"/>

    <!--  扫描SpringMVC的requestMapping 让处理url的路径配置生效，注解启动器 -->
    <mvc:annotation-driven/>

    <!--  重新映射静态资源路径，使得浏览器的url不需要在路径上增加static即可访问页面-->
    <mvc:resources mapping="/**" location="/static/"/>

<!--  设置自定义拦截器,页面在请求时首先进入拦截器，之后通过url  -->
    <mvc:interceptors>
        <mvc:interceptor>
            <!--  /**/*.html =>  /home_page.html  /abc/home_page.html    -->
            <!--  /*/*.html =>  /abc/home_page.html  无法匹配/home_page.html -->
            <mvc:mapping path="/*.html"/>
            <mvc:exclude-mapping path="/login.html"/>
            <bean class="club.banyuan.tools.AuthInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>

    <!--  c3p0连接池  -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${db.driver}"/>
        <property name="jdbcUrl" value="${db.url}"/>
        <property name="user" value="${db.username}"/>
        <property name="password" value="${db.password}"/>
    </bean>

    <!--创建数据工厂，通过工厂创建dao的代理对象-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
    <!--获取mapper的配置文件-->
        <property name="mapperLocations" value="classpath*:club/banyuan/dao/*.xml"/>
    <!--加载mybatis-config.xml文件-->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
    </bean>

    <!--扫描dao层的bean-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--    多个包路径使用,隔开-->
        <property name="basePackage" value="club.banyuan.dao.*"/>
    </bean>
```

```xml
<!--mybatis-config文件配置-->
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- 打印查询语句 -->
        <setting name="logImpl" value="STDOUT_LOGGING" />
    </settings>
</configuration>
```



