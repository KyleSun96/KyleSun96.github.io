---
title: SpringCloud整合练习（中）
date: 2020-05-24 23:00:50
tags: 
 - SpringCloud
 - 总结梳理
categories:
 - 总结梳理
comments: true
keywords: 
description: 
---

> SpringCloud整合练习（中）
>
> 图床待更新

<!-- more -->

# SpringCloud整合练习（中）

## 需求

![image-20200523140404041](img/image-20200523140404041.png)

1. 基于 Spring Boot 搭建商品搜索服务
   1. 商品关键字搜索(尽量符合用户的使用，可以参考京东)

2. 基于 RabbitMQ 的消息服务
   1. 商品服务添加新商品后，添加到 MySQL 中，并发送消息给 ES
   2. 搜索服务接收消息，并同步到 ES 中

## 商品服务发送消息

1. 导入amqp的起步依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-amqp</artifactId>
   </dependency>

   <dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>fastjson</artifactId>
       <version>1.2.68</version>
   </dependency>
   ```

2. 在application.yml配置mq的连接信息

   ```yml
   spring:
      rabbitmq:
        host: 192.168.200.129
        port: 5672
        virtual-host: /
        username: guest
        password: guest
   ```

3. 配置RabbitMQConfig类

   1. 创建queue
   2. 创建Exchange
   3. 绑定

   ```java
   @Configuration
    public class RabbitMQConfig {

        // 交换机名称
        public static final String GOODS_TOPIC_EXCHANGE = "goods_topic_exchange";
        // 队列名称
        public static final String GOODS_QUEUE = "goods_queue";

        // 声明交换机
        @Bean("goodsExchange")
        public Exchange topicExchange() {
            return ExchangeBuilder.topicExchange(GOODS_TOPIC_EXCHANGE).durable(true).build();
        }


        // 声明队列
        @Bean("goodsQueue")
        public Queue itemQueue() {
            return QueueBuilder.durable(GOODS_QUEUE).build();
        }


        // 绑定队列和交换机
        @Bean
        public Binding itemQueueExchange(@Qualifier("goodsQueue") Queue queue,
                                        @Qualifier("goodsExchange") Exchange exchange) {
            return BindingBuilder.bind(queue).to(exchange).with("goods.#").noargs();
        }

    }
   ```

4. GoodsService 中注入RabbitTempalte,发送消息

   ```java
    @Autowired
    RabbitTemplate rabbitTemplate;

    public void addGoods(Goods good) {

        // 将goods信息存入mysql
        goodsMapper.addGoods(good);

        // 准备信息内容
        /*
         当spec为map类型时，需要通过specStr转换

          String specStr = goods.getSpecStr();
          Map spec = JSON.parseObject(specStr,Map.class);
          goods.setSpec(spec);

          String data = JSON.toJSONString(goods);
          System.out.println(data);
         */
        String data = JSON.toJSONString(good);
        System.out.println(data);

        // 发送信息
        rabbitTemplate.convertAndSend(RabbitMQConfig.GOODS_TOPIC_EXCHANGE, "goods.add", data);
    }
   ```

## 搭建搜索服务

1. 添加依赖（web,amqp,es相关依赖）

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

        <!--引入es的坐标-->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>7.4.0</version>
        </dependency>

        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-client</artifactId>
            <version>7.4.0</version>
        </dependency>

        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.4.0</version>
        </dependency>

        <!-- eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>

        <!-- hystrix -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>

        <!--json-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.68</version>
        </dependency>

    </dependencies>

   ```

2. 启动类

   ```java
    @EnableEurekaClient
    @SpringBootApplication
    public class ElasticSearchApp {

        public static void main(String[] args) {
            SpringApplication.run(ElasticSearchApp.class, args);
        }

    }

   ```

3. 配置文件

  application.yml

   ```yml
    server:
      port: 9003

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
        name: business-elasticsearch # 设置当前应用的名称。将来会在eureka中Application显示。将来需要使用该名称来获取路径

      rabbitmq:
        host: 192.168.200.129
        port: 5672
        virtual-host: /
        username: guest
        password: guest

    elasticsearch:
      host: 192.168.200.129
      port: 9200

   ```

## 搜索服务接收消息

1. 复制消息生产者的配置类

   ![image-20200523144117956](img/image-20200523144117956.png)

2. 编写消息的监听类

   ```java
    @Component
    public class GoodsListener {

        @Autowired
        private ESService esService;

        /**
        * @description: //TODO 监听指定队列的消息，收到消息，并且添加文档
        * @param: [jsonGoods]
        * @return: void
        */
        @RabbitListener(queues = RabbitMQConfig.GOODS_QUEUE)
        public void goodsListener(String jsonGoods) {
            System.out.println("接收到的消息为：" + jsonGoods);

        }

    }
   ```

3. 测试消息

   ![image-20200523144504018](img/image-20200523144504018.png)

## 搜索服务添加ES文档

1. 搭建ES环境，创建索引和映射

   ```json
    PUT goods
    {
        "mappings": {
            "properties": {
                "title": {
                    "type": "text",
                    "analyzer": "ik_smart"
                },
                "price": {
                    "type": "double"
                },
                "createTime": {
                    "type": "date"
                },
                "categoryName": {
                    "type": "keyword"
                },
                "brandName": {
                    "type": "keyword"
                },
                "spec": {
                    "type": "text"
                },
                "saleNum": {
                    "type": "integer"
                },
                "stock": {
                    "type": "integer"
                },
                "seller":{
                    "type": "keyword"
                },
                "company":{
                    "type": "keyword"
                }
            }
        }
    }
   ```

2. 创建 ES 高级客户端配置类

   ```java
    @Configuration
    @ConfigurationProperties(prefix = "elasticsearch")
    public class ElasticSearchConfig {

        @Value("${elasticsearch.host}")
        private String host;

        @Value("${elasticsearch.port}")
        private int port;

        @Bean
        public RestHighLevelClient client() {
            return new RestHighLevelClient(RestClient.builder(
                    new HttpHost(host, port, "http")
            ));
        }
    }
   ```

3. 编写 service 同步数据库 和 保存文档信息 方法

   ```java
    @Service
    public class ESService {

        @Autowired
        private ESMapper esMapper;

        @Autowired
        private RestHighLevelClient client;

        /**
          * @description: //TODO 同步所有数据
          * @param: []
          * @return: void
          */
        public void importAll() throws IOException {

            List<Goods> goodsList = esMapper.findAll();

            // bulk导入
            BulkRequest bulkRequest = new BulkRequest();

            // 循环goodsList，创建IndexRequest添加数据
            for (Goods goods : goodsList) {

                // 给同步到 ES 中的数据添加 ID，ID值为 mysql 中的 ID
                double goodsId = goods.getId();

                // 将 goods对象转换为json字符串
                String data = JSON.toJSONString(goods);

                IndexRequest indexRequest = new IndexRequest("goods").id(goodsId + "").source(data, XContentType.JSON);
                bulkRequest.add(indexRequest);
            }

            BulkResponse response = client.bulk(bulkRequest, RequestOptions.DEFAULT);
            // 同步所有数据状态：是否成功
            System.out.println(response.status());
        }


        /**
          * @description: //TODO 添加单个商品后，同步到 ES
          * @param: [jsonGoods]
          * @return: void
          */
        public void addGoods(String jsonGoods) throws IOException {

            // 转为 Goods 对象，拿到 id
            Goods goods = JSON.parseObject(jsonGoods, Goods.class);

            IndexRequest request = new IndexRequest("goods").id(goods.getId() + "").source(jsonGoods, XContentType.JSON);

            IndexResponse response = client.index(request, RequestOptions.DEFAULT);

        }
   ```

4. 在listener进行调用

   ```java
   @Component
    public class GoodsListener {

        @Autowired
        private ESService esService;

        /**
        * @description: //TODO 监听指定队列的消息，收到消息，并且添加文档
        * @param: [jsonGoods]
        * @return: void
        */
        @RabbitListener(queues = RabbitMQConfig.GOODS_QUEUE)
        public void goodsListener(String jsonGoods) {
            System.out.println("接收到的消息为：" + jsonGoods);

            // 收到消息，并且添加文档到 ES
            try {
                esService.addGoods(jsonGoods);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

    }
   ```

5. 测试

   问题：传到ES消息内容中goods没有id，因此添加ES文档操作变成了覆盖。

   解决：消息中应该将goods id添加到信息中，goods的id是由mysql自增长，所以在GoodsMapper中insert中需要配置返回主键。

   ```java
    @Mapper
    public interface GoodsMapper {

        @Insert("INSERT INTO `goods` ( `title`, `price`, `stock`, `saleNum`, `createTime`, `categoryName`, `brandName`, `spec`, `seller`, `company` )\n" +
                            "VALUES(#{title},#{price},#{stock},#{saleNum},#{createTime},#{categoryName},#{brandName},#{spec},#{seller},#{company})")
        @Options(keyProperty = "id", useGeneratedKeys = true)
        public void addGoods(Goods good);

    }
   ```

## 搜索服务商品查询

1. 需求分析

    1. 如果没有输入查询关键字 keyword ,则不进行查询
    2. 使用 query String 对 keyword 进行查询，从 title,brandName,category 进行查询，操作 OR
    3. 添加 brandName 条件，filter
    4. 添加价格，最大值最小值：price=1000-2000

2. 设计

   参数 Map

   返回 List

3. 编写service，商品查询

   ```java
    /**
     * @description: //TODO 商品查询
     * @param: [param]
     * @return: java.util.List<com.itheima.domain.Goods>
     * <p>
     * 1. 如果没有输入查询关键字 keyword ,则不进行查询
     * 2. 使用 query String 对 keyword 进行查询，从 title,brandName,category 进行查询，操作 OR
     * 3. 添加 brandName 条件，filter
     * 4. 添加价格，最大值最小值：price=1000-2000
     */
    public List<Goods> findGoods(Map<String, String> param) throws IOException {
        SearchRequest searchRequest = new SearchRequest("goods");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();

        BoolQueryBuilder query = QueryBuilders.boolQuery();

        // 如果没有输入查询关键字 keyword ,则不进行查询
        if (param == null || StringUtils.isBlank(param.get("keyword"))) {
            return new ArrayList<>();
        }

        // 使用 query String 对 keyword 进行查询，从 title,brandName,category 进行查询，操作 OR
        QueryStringQueryBuilder queryStringQuery = QueryBuilders.queryStringQuery(param.get("keyword"))
                .field("title").field("categoryName").field("brandName")
                .defaultOperator(Operator.OR);
        query.must(queryStringQuery);

        // 添加 brandName 条件，filter
        if (StringUtils.isNotBlank(param.get("brandName"))) {
            TermQueryBuilder termQuery = QueryBuilders.termQuery("brandName", param.get("brandName"));
            query.filter(termQuery);
        }

        // 添加价格，最大值最小值：price = 1000-2000
        if (StringUtils.isNotBlank(param.get("price"))) {
            String[] priceArr = param.get("price").split("-");

            RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("price");
            rangeQuery.gte(String.valueOf(priceArr[0]));
            rangeQuery.lte(String.valueOf(priceArr[1]));

            query.filter(rangeQuery);
        }

        // 使用 boolQuery 连接
        sourceBuilder.query(query);

        searchRequest.source(sourceBuilder);

        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        SearchHits searchHits = searchResponse.getHits();
        long total = searchHits.getTotalHits().value;
        System.out.println("总记录数：" + total);

        List<Goods> goodsList = new ArrayList<>();
        SearchHit[] hits = searchHits.getHits();
        for (SearchHit hit : hits) {
            String sourceAsString = hit.getSourceAsString();
            Goods goods = JSON.parseObject(sourceAsString, Goods.class);
            goodsList.add(goods);
        }
        return goodsList;
    }
   ```

4. controller

   ```java
    @RestController
    @RequestMapping("/search")
    public class ESController {

        @Autowired
        private ESService esService;

        /**
        * @description: //TODO 同步所有数据
        * @param: []
        * @return: com.itheima.entity.Result
        */
        @RequestMapping("/importAll")
        public Result importAll() {
            try {
                esService.importAll();
                return new Result(true, "同步所有数据成功");
            } catch (Exception e) {
                e.printStackTrace();
                return new Result(false, "同步所有数据失败");
            }
        }


        /**
        * @description: //TODO 商品查询
        * @param: [query]
        * @return: com.itheima.entity.Result
        */
        @PostMapping("/findGoods")
        public Result findGoods(@RequestBody Map<String, String> param) {
            try {
                List<Goods> goodsList = esService.findGoods(param);
                return new Result(true, "商品查询成功", goodsList);
            } catch (Exception e) {
                e.printStackTrace();
                return new Result(false, "商品查询失败");
            }
        }

    }
   ```

5. 测试

## 在商家服务中调用商品搜索

1. 在商家服务中编写feign客户端
2. 测试商家服务可以远程调用搜索服务

   ```java
    @FeignClient(value = "business-elasticsearch")
    public interface SearchFeignClient {

        @PostMapping("/search/findGoods")
        public Result findGoods(@RequestBody Map<String, String> param);

    }

   ```

## postman 中测试数据

1. 测试调用端 商家服务添加商品
   1. 测试URL：<http://localhost:9001/seller/addGoods/1>
   2. 测试数据：

   ```json
    {
    "title":"荣耀X10 5G双模 麒麟820 4300mAh续航 4000万高感光影像系统 6.63英寸升降全面屏 全网通6GB+128GB 竞速蓝",
    "price":4999,
    "saleNum":6521,
    "categoryName":"手机",
    "brandName":"华为"
    }
   ```

    ![测试](img/20200524225242.png)

2. 测试商家服务同步数据中的数据到 ES
   1. 测试URL：<http://localhost:9003/search/importAll>

3. 测试查询商品
   1. ES 搜索端测试URL：<http://localhost:9003/search/findGoods>
   2. 商家服务端远程测试URL：<http://localhost:9001/seller/findGoods>
   3. 测试数据：

   ```json
    {
    "keyword":"手机",
    "brandName":"华为",
    "price":"1000-5000"
    }
   ```
