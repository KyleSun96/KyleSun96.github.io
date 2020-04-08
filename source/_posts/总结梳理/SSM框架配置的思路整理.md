---
title: SSM框架配置的思路整理
date: 2020-03-28 15:21:59
tags: 
 - SSM
 - 总结梳理
categories:
 - 总结梳理
comments: true
keywords: 
description: 

---

# 配置文件的思路及整理


- 配置web.xml 用于将Spring和SpringMVC框架集成到Web项目中

***


### 1. web.xml

1. Spring部分

    - 声明spring的监听器ContextLoaderListener,用于加载spring的核心配置文件
	  其内部帮我们加载配置文件,并创建Spring容器,因此需要配置一个全局的初始化参数

    - 声明全局初始化参数contextConfigLocation,加载配置文件applicationContext.xml
2. SpringMVC部分

    - 声明SpringMVC的前端控制器DispatcherServlet,用于加载SpringMVC的核心配置文件spring-mvc.xml
3. 配置统一编码过滤器

    - 配置 编码过滤器 用以解决Tomcat8 post请求方式请求乱码问题

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="3.0" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">

    <!--配置统一编码过滤器-->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--配置监听器  全局参数-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!--配置前端控制器-->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>


</web-app>

```

***


### 2. spring-mvc.xml

1. 配置包注解扫描
2. 配置注解驱动
    - 加载处理器映射器 RequestMappingHandlerMapping
    - 加载处理器适配器 RequestMappingHandlerAdapter
    - 内部集成了JackSon,自动将对象或集合转为Json
3. 开启静态资源访问
4. 配置内部资源视图解析器
5. 配置简单统一异常处理器
6. 配置登录权限拦截器

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">


    <!--注解扫描包-->
    <context:component-scan base-package="com.company.controller"/>

    <!--注解驱动-->
    <mvc:annotation-driven/>

    <!--开启静态资源访问-->
    <mvc:default-servlet-handler/>

    <!--内部视图解析器-->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/pages/"/>
        <property name="suffix" value=".jsp"/>
    </bean>

    <!--异常统一处理-->
    <!--
        <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
            <property name="defaultErrorView" value="redirect:/500.jsp"/>
        </bean>
    -->

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <mvc:exclude-mapping path="/user/login"/>
            <bean class="com.company.interceptor.PrivilegeInterceptor"/>
        </mvc:interceptor>
    </mvc:interceptors>

</beans>
```

***


### 3. applicationContext.xml

1. 开启包注解扫描
2. 引入外部数据库配置文件
3. 配置数据源
4. 配置SqlSessionFactoryBean
5. 扫描mapper所在的包 为mapper创建实现类
6. 配置声明式事务管理
7. 配置事务控制
8. 事务织入: 使用事务控制 对 类的指定方法进行增强

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--开启注解扫描-->
    <context:component-scan base-package="com.company">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--引入外部数据库配置文件-->
    <context:property-placeholder location="classpath:jdbc.properties"/>

    <!--配置数据源-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    
	<!--整合Mybatis-->
    <!--创建SqlSessionFactory 同时指定数据源-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--加载核心配置文件-->
        <property name="configLocation" value="classpath:sqlMapConfig.xml"/>
    </bean>

    <!--扫描mapper所在的包 为mapper创建实现类-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.company.mapper"/>
    </bean>


    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置声明式事务控制-->
    <tx:advice id="txAdvice">
        <tx:attributes>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="save*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="add*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="update*" propagation="REQUIRED" read-only="false"/>
            <tx:method name="delete*" propagation="REQUIRED" read-only="false"/>
        </tx:attributes>
    </tx:advice>

    <!--配置织入-->
    <aop:config>
        <aop:pointcut id="pc" expression="execution(* com.company.service.impl.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pc"></aop:advisor>
    </aop:config>


</beans>
```

***

### 4.sqlMapConfig.xml

mybatis的核心配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <typeAliases>
        <!--
            统一设置别名: 类名就是别名,不区分大小写
        -->
        <package name="com.company.domain"/>
    </typeAliases>

    <!--分页助手-->
    <plugins>
        <plugin interceptor="com.github.pagehelper.PageHelper">
            <!--配置参数:配置数据库方言-->
            <property name="dialect" value="mysql"/>
        </plugin>
    </plugins>
</configuration>
```

