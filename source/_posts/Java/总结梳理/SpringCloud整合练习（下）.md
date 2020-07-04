---
title: SpringCloud整合练习（下）
date: 2020-05-26 21:50:55
tags: 
 - SpringCloud
 - 总结梳理
categories:
 - 总结梳理
comments: true
keywords: 
description: 
---
> SpringCloud整合练习（下）
>
> 图床待更新

<!-- more -->

# SpringCloud整合练习（下）

## 需求

1. 搭建网关gateway，配置路由
2. 搭建注册中心 config server
3. 将配置信息上传上gitee
4. 将商家服务，商品服务，搜索服务的信息配置到配置中心
5. 配置bus一键刷新
6. 使用docker部署商家服务，商品服务，搜索服务

## 网关的配置

操作步骤：

1. 搭建网关模块，导入资料中的初始化代码

   其实就是创建一个最基本的spring cloud模块。

2. 引入依赖：starter-gateway

   ```xml
   <dependencies>
      <!--引入gateway 网关-->
   <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-gateway</artifactId>
      </dependency>

   <!-- eureka-client -->
      <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
      </dependency>
   </dependencies>
   ```

3. 编写启动类,无特殊操作

   ```java
   @EnableEurekaClient
   @SpringBootApplication
   public class GatewayApp {

       public static void main(String[] args) {
           SpringApplication.run(GatewayApp.class, args);
       }

   }
   ```

4. 编写配置文件

   ```yml
   server:
     port: 80

   eureka:
     client:
       service-url:
         defaultZone: http://localhost:8761/eureka

   spring:
     application:
       name: business-gateway

     cloud:
       # 网关配置
       gateway:
         # 路由配置：转发规则
         routes: #集合
         # id: 唯一标识。默认是一个UUID
         # uri: 转发路径
         # predicates: 条件,用于请求网关路径的匹配规则

         - id: gateway-seller
           uri: lb://business-seller
           predicates:
           - Path=/seller/**

         - id: gateway-goods
           uri: lb://business-goods
           predicates:
           - Path=/goods/**

         # 微服务名称配置(非必须)
         discovery:
           locator:
             enabled: true # 设置为true 请求路径前可以添加微服务名称
             lower-case-service-id: true # 允许为小写
   ```

5. 测试

## 配置中心服务

### 搭建配置中心服务

1. 准备环境，创建git仓库

   ![image-20200527143355109](img/image-20200527143355109.png)

2. 在本地克隆git仓库，管理配置文件

   **注意：**除了注册中心意以外的信息，都可以通过配置中心进行管理。

   ![image-20200527144224570](img/image-20200527144224570.png)

3. 搭建配置中心

   1. 添加依赖

      ```xml
      <dependencies>
         <!-- config-server -->
         <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-config-server</artifactId>
         </dependency>

         <!-- eureka-client -->
         <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
         </dependency>
      </dependencies>
      ```

4. 启动类

   ```java
   @EnableEurekaClient
   @EnableConfigServer // 启用config server功能
   @SpringBootApplication
   public class ConfigServerApp {

       public static void main(String[] args) {
           SpringApplication.run(ConfigServerApp.class,args);
       }

   }
   ```

5. 配置文件

   ```yml
   server:
     port: 9527

   # 将自己注册到eureka中
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:8761/eureka

   spring:
     application:
       name: config-server
     # spring cloud config
     cloud:
    config:
         server:
           git:         # git 的 远程仓库地址
             uri: https://gitee.com/kylesun96/config.git
         label: master  # 分支配置
   ```

6. 测试

   <http://localhost:9527/master/seller-dev.yml>

### 配置中心客户端

1. 在客户端模块 business-seller 添加依赖

   ```xml
   <!--config client -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
   ```

2. 在客户端模块 business-seller，添加配置文件 bootstrap.yml

   ```yml
   # 配置config-server地址
   # 配置获得配置文件的名称等信息
   spring:
     cloud:
       config:
         # 配置config-server地址
         # uri: http://localhost:9527
         # 配置获得配置文件的名称等信息
         name: seller # 文件名
         profile: dev # profile指定，  seller-dev.yml
         label: master # 分支
         # 从注册中心寻找config-server地址
         discovery:
           enabled: true
           service-id: config-server

   eureka:
     client:
       service-url:
         defaultZone: http://localhost:8761/eureka

   ```

### BUS全局刷新——配置文件修改后无需重启即可生效

#### 1. 单个应用热部署， 如：business-seller 

config client刷新操作步骤：

1. 在 seller 客户端引入 actuator 依赖

   ```xml
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. 获取配置信息类上，添加 @RefreshScope 注解

   ```java
   @RestController
   @RefreshScope // 开启刷新功能
   @RequestMapping("/seller")
   public class SellerController {
   ```

3. 添加 bootstrap.yml 配置
   management.endpoints.web.exposure.include: refresh

   ```yml
   # 用于暴露seller的刷新端点
   management:
     endpoints:
       web:
         exposure:
           include: '*'     # 暴漏的endpoint，* 表示所有

   ```

4. 修改配置文件，如：更改数据源，并提交到远程仓库。

   <img src="img/image-20200527200029697.png" alt="image-20200527200029697" style="zoom:80%;" />

5. 使用 curl工具/postman 发送post请求，热部署成功后显示修改了哪些信息
   curl -X POST http://localhost:9001/actuator/refresh

   <img src="img/image-20200527200501195.png" alt="image-20200527200501195" style="zoom:67%;" />

6. 输入相同的地址验证数据：

   <http://localhost:9001/seller/findById/1>

   左图为热部署前获取本机中数据库的数据

   右图为热部署后获取虚拟机docker中数据库的数据

   ![image-20200527201230603](img/image-20200527201230603.png)

#### 2. 通过BUS进行统一热部署

1. 分别在 business-config 和 business-seller 中引入 bus依赖：bus-amqp

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>

   <!-- bus -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-bus-amqp</artifactId>
   </dependency>
   ```

2. 分别在 config-server 和 config-client中配置 RabbitMQ

   bootstrap.yml 和 config-server 的 application.yml

   ```yml
   spring:
      # 配置rabbitmq信息
      rabbitmq:
         host: 192.168.200.129
         port: 5672
         username: guest
         password: guest
         virtual-host: /
   ```

3. 在 config-server 中设置暴露监控断点：bus-refresh

   通过 actuator 来实现，添加 actuator 依赖（已添加）

   ```xml
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

   暴漏endpoint的配置

   ```yml
   # 用于暴露bus的刷新端点
   management:
     endpoints:
       web:
         exposure:
           include: 'bus-refresh'
   ```

4. 启动测试

   访问的是config-server的刷新节点：

   curl -X POST <http://localhost:9527/actuator/bus-refresh>

   具体步骤参考单个应用热部署的图片。

## 项目部署上线

1. 项目打包

   springboot项目打jar包，添加打包插件。

   需要打包的工程中的添加

   ```xml
   <build>
      <plugins>
         <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
         </plugin>
      </plugins>
   </build>
   ```

2. 执行maven打包命令  package

3. 从target中，复制打包好的jar，然后进行运行

      java -jar  fuxi_seller-1.0-SNAPSHOT.jar

      问题：

      打包失败，找不到pojo工程。

      解决方法： clean，并install应用。

4. jar包部署

5. docker容器部署

   springboot应用，要连接docker中运行的mysql容器，那么数据源的配置应该使用 宿主机的IP和映射的端口。

   ![image-20200527162420183](img/image-20200527162420183.png)

   1. dockfile制作镜像

      1. 修改数据源信息，并打包应用。

      2. 编写dockerfile文件

         springboot_dockerfile

         ```text
         FROM java:8
         MAINTAINER itheima<itheima@itcast.cn>

         ADD fuxi_seller-1.0-SNAPSHOT.jar app.jar
         CMD java -jar app.jar
         ```

         ![image-20200527163146251](img/image-20200527163146251.png)

      3. 将打包好的jar和springboot_dockerfile上传到服务器上

         在虚拟机中创建目录

         ```shell
         cd ~

         mkdir apps

         cd apps
         ```

         CRT中打开上传fstp.alt+p

         ```shell
         #在windows下，进入有jar和dockfile文件的目录（不能带中文）
         lcd C:\Users\admin\Desktop\apps

         #进服务器，进入上面新建的apps目录中
         cd apps

         #上传文件
         put fuxi_seller-1.0-SNAPSHOT.jar
         put springboot_dockerfile
         ```

      4. 根据上传的文件创建docker镜像

         ```shell
         docker build -f ./springboot_dockerfile -t itheima_app:1 .
         ```

         ![image-20200527164007840](img/image-20200527164007840.png)

      5. 根据镜像启动容器

         ```sh
         docker run -id --name=c_springboot -p 9001:9001 itheima_app:1
         ```

      6. 测试

         ![image-20200527164231192](img/image-20200527164231192.png)
