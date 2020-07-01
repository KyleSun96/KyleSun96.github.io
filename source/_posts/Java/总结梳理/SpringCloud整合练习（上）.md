---
title: SpringCloud整合练习（上）
date: 2020-05-22 14:56:50
tags: 
 - SpringCloud
 - 总结梳理
categories:
 - 总结梳理
comments: true
keywords: 
description: 
---

> 在学习过 SpringBoot，Docker，RabbitMQ，SpringCloud，ElasticSearch的零散知识点后，对于各知识点没有统一连贯的把握
>
> 因此做一个小练习，用于巩固SpringBoot和SpringCloud中学习到的知识点，以加深对SpringBoot以及SpringCloud的理解和使用
>
> 练习代码 [点击此处](https://github.com/KyleSun96/springcloud-learning-samples) 其中的part08_springcloud_practice 部分
>
> 图床待更新

<!-- more -->

# SpringCloud整合练习（上）

## 需求

![需求](img/image-20200522140630313.png)

1. 基于 Spring Boot 搭建商家服务，商品服务

2. 搭建 Eureka Server 注册中心

3. 用户服务

   1. 根据ID查询商家信息
   2. 添加商品信息（调用商品服务）

4. 商品服务

   1. 添加商品信息到MySQL：测试数据

      ```json
      {
       "title":"荣耀X10 5G双模 麒麟820 4300mAh续航 4000万高感光影像系统 6.63英寸升降全面屏 全网通6GB+128GB 竞速蓝",
       "price":2199.00,
       "saleNum":6521,
       "categoryName":"手机",
       "brandName":"华为"
      }
      ```

5. 使用 Hystix 对添加商品进行降级处理

## 搭建环境

1. 数据库

   ![Snipaste_2020-05-22_16-35-39.png](img/Snipaste_2020-05-22_16-35-39.png)

2. 创建SpringCloud的父工程

   pom.xml

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>

       <groupId>com.itheima</groupId>
       <artifactId>part08_springcloud_practice</artifactId>
       <packaging>pom</packaging>
       <version>1.0-SNAPSHOT</version>

       <!--spring Boot 环境 -->
       <parent>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-parent</artifactId>
           <version>2.1.8.RELEASE</version>
           <relativePath/>
       </parent>

       <properties>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
           <java.version>1.8</java.version>
           <!--Spring Cloud版本-->
           <spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
       </properties>

       <!--引入 Spring Cloud 依赖-->
       <dependencyManagement>
           <dependencies>
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>${spring-cloud.version}</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
           </dependencies>
       </dependencyManagement>

       <dependencies>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <version>1.18.10</version>
               <scope>provided</scope>
           </dependency>
       </dependencies>

    </project>

    ```

3. 创建公用的pojo模块，封装JavaBean

   ![Snipaste_2020-05-22_16-30-18.png](img/Snipaste_2020-05-22_16-30-18.png)

   注意： 如果需要使用流来传输对象，需要实现IO包的序列化接口Serializable。**一般默认加上。**

## 搭建注册中心

步骤：

1. 创建 eureka-server 模块

    ```xml
    <!-- 继承父工程 -->
    <parent>
        <artifactId>spring-cloud-parent</artifactId>
        <groupId>com.itheima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    ```

2. 引入 SpringCloud 和 euraka-server 相关依赖

    ```xml
    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- eureka-server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

    </dependencies>
    ```

3. 完成 Eureka Server 相关配置

    ```yml
    server:
    port: 8761

    # eureka 配置
    # eureka 一共有4部分 配置
    # 1. dashboard:eureka的web控制台配置
    # 2. server:eureka的服务端配置
    # 3. client:eureka的客户端配置
    # 4. instance:eureka的实例配置

    eureka:
    instance:
        hostname: localhost # 主机名
    client:
        service-url:
          defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka # eureka服务端地址，将来客户端使用该地址和eureka进行通信
        register-with-eureka: false # 是否将自己的路径 注册到eureka上。eureka server 不需要的，eureka provider client 需要
        fetch-registry: false # 是否需要从eureka中抓取路径。eureka server 不需要的，eureka consumer client 需要
    server:
        enable-self-preservation: false # 关闭自我保护机制
        eviction-interval-timer-in-ms: 3000 # 检查服务的时间间隔

    ```

4. 启动该模块

    ```java
    @EnableEurekaServer // 启用EurekaServer
    @SpringBootApplication
    public class EurekaApp {

        public static void main(String[] args) {
            SpringApplication.run(EurekaApp.class,args);

        }

    }

    ```

## 搭建商家服务

1. 创建seller模块

2. 基于Spring Boot搭建SSM模块

   1). 添加依赖，配置pom.xml

    ```xml
       <dependencies>

            <dependency>
                <groupId>com.itheima</groupId>
                <artifactId>business-common</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>

            <!--spring boot web-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-web</artifactId>
            </dependency>

            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.1.0</version>
            </dependency>

            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
            </dependency>

            <!-- eureka-client -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            </dependency>

        </dependencies>
    ```

   2). 编写启动类

    ```java
        @EnableEurekaClient
        @SpringBootApplication
        public class SellerApp {

            public static void main(String[] args) {
            SpringApplication.run(SellerApp.class,args);
            }
        }
    ```

   3). 编写配置文件

    ```yml
    server:
        port: 9001

      eureka:
        instance:
          hostname: localhost # 主机名
        client:
          service-url:
            defaultZone: http://localhost:8761/eureka # eureka服务端地址，将来客户端使用该地址和eureka进行通信

      spring:
        datasource:
          url: jdbc:mysql:///db_user?serverTimezone=Asia/Shanghai
          username: root
          password: root
          driver-class-name: com.mysql.cj.jdbc.Driver

        application:
          name: business-seller # 设置当前应用的名称。将来会在eureka中Application显示。将来需要使用该名称来获取路径
    ```

## 根据ID查询商家信息

1. mapper

   ```java
   @Mapper
   public interface SellerMapper {

     @Select("select * from seller where id = #{id}")
     public Seller findById(int id);

   }

   ```

2. service

   ```java
   @Service
   public class SellerService {

       @Autowired
       private SellerMapper sellerMapper;

       public Seller findById(int id) {
           return sellerMapper.findById(id);
       }

   }
   ```

3. controller

   ```java
   @RestController
   @RequestMapping("/seller")
   public class SellerController {

       @Autowired
       SellerService sellerService;

       @GetMapping("/findById/{id}")
       public Result findById(@PathVariable("id") int id) {
           try {
               Seller seller = sellerService.findById(id);
               return new Result(true, "查询商家信息成功", seller);
           } catch (Exception e) {
               e.printStackTrace();
               return new Result(false, "查询商家信息失败");
           }
       }
   }
   ```

4. 测试

## 搭建商品服务

1. 创建goods模块

2. 基于Spring Boot搭建SSM模块

   1). 添加依赖，配置pom.xml

    ```xml
    <dependencies>
        <dependency>
            <groupId>com.itheima</groupId>
            <artifactId>business-common</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>

        <!--spring boot web-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.0</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>

        <!-- eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

    </dependencies>
    ```

   2). 编写启动类

    ```java
      @EnableEurekaClient
      @SpringBootApplication
      public class GoodsApp {

        public static void main(String[] args) {
            SpringApplication.run(GoodsApp.class,args);
        }

      }
    ```

   3). 编写配置文件

      ```yml
      server:
        port: 9002

      eureka:
        instance:
          hostname: localhost # 主机名
        client:
          service-url:
            defaultZone: http://localhost:8761/eureka # eureka服务端地址，将来客户端使用该地址和eureka进行通信

      spring:

        datasource:
          url: jdbc:mysql:///db_goods?serverTimezone=Asia/Shanghai
          username: root
          password: root
          driver-class-name: com.mysql.cj.jdbc.Driver

        application:
          name: business-goods # 设置当前应用的名称。将来会在eureka中Application显示。将来需要使用该名称来获取路径
      ```

## 提供方-商品添加

1. mapper

   ```java
   @Mapper
   public interface GoodsMapper {

       @Insert("INSERT INTO `goods` ( `title`, `price`, `stock`, `saleNum`, `createTime`, `categoryName`, `brandName`, `spec`, `seller`, `company` )\n" +
                             "VALUES(#{title},#{price},#{stock},#{saleNum},#{createTime},#{categoryName},#{brandName},#{spec},#{seller},#{company})")
       public void addGoods(Goods good);

   }
   ```

2. service

   ```java
   @Service
   public class GoodsService {

       @Autowired
       private GoodsMapper goodsMapper;

       public void addGoods(Goods good) {
           goodsMapper.addGoods(good);
       }
   }
   ```

3. controller

   ```java
   @RestController
   @RequestMapping("/goods")
   public class GoodsController {

       @Autowired
       private GoodsService goodsService;

       @PostMapping("/add")
       public Result addGoods(@RequestBody Goods good) {
           goodsService.addGoods(good);
           return new Result(true, "添加商品信息成功！");
       }

   }
   ```

4. 测试

   ```json
   {
    "title":"荣耀X10 5G双模 麒麟820 4300mAh续航 4000万高感光影像系统 6.63英寸升降全面屏 全网通6GB+128GB 竞速蓝",
    "price":2199.00,
    "saleNum":6521,
    "categoryName":"手机",
    "brandName":"华为"
    }
   ```

## 调用方-商品添加

1. 添加feign依赖

   ```xml
   <!--feign-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   ```

2. 启动feign

   ```java
   @EnableFeignClients    // 开启Feign的功能
   @EnableEurekaClient
   @SpringBootApplication
   public class SellerApp {

       public static void main(String[] args) {
           SpringApplication.run(SellerApp.class, args);
       }

   }
   ```

3. 编写Feign客户端，配置

   ```java
   @FeignClient(value = "business-goods")
   public interface GoodsFeignClient {

       @PostMapping("/goods/add")
       public Result addGoods(@RequestBody Goods good);

   }
   ```

4. 在controller中，调用feign客户端

   ```java

       @Autowired
       private GoodsFeignClient goodsFeignClient;

       @PostMapping("/addGoods/{sellerId}")
       public Result addGoods(@PathVariable("sellerId") int sellerId, @RequestBody Goods good) {

           Seller seller = sellerService.findById(sellerId);
           good.setSeller(seller.getSeller());
           good.setCompany(seller.getCompany());
           good.setCreateTime(new Date());
           good.setSpec("{\"机身内存\":\"16G\",\"网络\":\"联通3G\"}");

           Result result = goodsFeignClient.addGoods(good);
           return result;

       }
   ```

5. 测试

   ```json
   {
    "title":"荣耀X10 5G双模 麒麟820 4300mAh续航 4000万高感光影像系统 6.63英寸升降全面屏 全网通6GB+128GB 竞速蓝",
    "price":2199.00,
    "saleNum":6521,
    "categoryName":"手机",
    "brandName":"华为"
   }
   ```

## Hystix 降级处理-服务提供方

操作步骤：

1. 在服务提供方，引入 hystrix 依赖

   ```xml
   <!-- hystrix -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
   </dependency>
   ```

2. 在启动类上开启Hystrix功能：@EnableCircuitBreaker

   ```java
   @EnableCircuitBreaker  // 开启Hystrix功能
   @EnableEurekaClient
   @SpringBootApplication
   public class GoodsApp {
   ```

3. 定义降级方法

   ```java
      /**
        * 定义降级方法：
        * 1. 方法的返回值需要和原方法一样
        * 2. 方法的参数需要和原方法一样
        * 3. 就是只有方法名不同
        */
       public Result addGoods_fallback(@RequestBody Goods good) {
           return new Result(false, "服务提供方降级！");
       }
   ```

4. 使用 @HystrixCommand 注解配置降级方法

   ```java
   @PostMapping("/add")
   @HystrixCommand(fallbackMethod = "addGoods_fallback")
   public Result addGoods(@RequestBody Goods good) {

       // 模拟超时异常
        /*try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }*/

       // 模拟提供端异常
       if (good.getPrice() < 100) {
           throw new RuntimeException("价格不符，无法添加！");
       }

       goodsService.addGoods(good);
       return new Result(true, "添加商品信息成功！");
   }
   ```

## Hystix 降级处理-调用方

操作步骤：

1. feign 组件已经集成了 hystrix 组件。

   ![Snipaste_2020-05-22_17-47-11.png](img/Snipaste_2020-05-22_17-47-11.png)

2. 在application.yml中配置开启 feign.hystrix.enabled = true

   ```yml
   # 开启feign对hystrix的支持
   feign:
     hystrix:
       enabled: true
   ```

3. 定义 feign 调用接口实现类，复写方法，即 降级方法

   ```java
   /**
    * Feign 客户端的降级处理类
    * 1. 定义类 实现 Feign 客户端接口
    * 2. 使用@Component注解将该类的Bean加入SpringIOC容器
    */
   @Component
   public class GoodsFeignClientFallback implements GoodsFeignClient {
       @Override
       public Result addGoods(Goods good) {
           return new Result(false,"服务调用方降级！");
       }
   }
   ```

4. 在 @FeignClient 注解中使用 fallback 属性设置降级处理类。

   ```java
   @FeignClient(value = "business-goods", fallback = GoodsFeignClientFallback.class)
   public interface GoodsFeignClient {

       @PostMapping("/goods/add")
       public Result addGoods(@RequestBody Goods good);

   }
   ```

5. 测试

   ```json
   {
    "title":"荣耀X10 5G双模 麒麟820 4300mAh续航 4000万高感光影像系统 6.63英寸升降全面屏 全网通6GB+128GB 竞速蓝",
    "price":2199.00,
    "saleNum":6521,
    "categoryName":"手机",
    "brandName":"华为"
   }
   ```
