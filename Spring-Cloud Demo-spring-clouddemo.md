---
title: Spring-Cloud Demo
date: 2021-07-28 09:24:01.0
updated: 2022-04-27 16:35:24.302
url: /archives/spring-clouddemo
categories: 
- Java
tags: 
- Spring-Cloud
---



# Spring-Cloud 使用步骤
[TOC]


## 一、 使用到 的工具

- Eureka

- Fegin

- 消费者 服务者

## 二、搭建步骤
## 2.1 搭建 微服务父工程
> pom文件如下
```
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.peng</groupId> 
    <artifactId>Spring-Cloud-Bill</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>Spring-Cloud-Bill</name>
    <packaging>pom</packaging>
    <modules>
        <module>customer-service</module>
        <module>user-service</module>
        <module>Eureka-service</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
    </parent>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Greenwich.SR1</spring-cloud.version>
        <mapper.starter.version>2.1.5</mapper.starter.version>
        <mysql.version>5.1.46</mysql.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <!-- springCloud -->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <!-- 通用Mapper启动器 -->
            <dependency>
                <groupId>tk.mybatis</groupId>
                <artifactId>mapper-spring-boot-starter</artifactId>
                <version>${mapper.starter.version}</version>
            </dependency>
            <!-- mysql驱动 -->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
            </dependency>
        </dependencies>
    </dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
```
 上述代码中 使用SpringBoot 启动器作为 父工程，添加子工程 module

 ### 2.2 搭建 Eureka 注册中心
 #### 2.2.1 pom 文件依赖关系
 ```
 # 设定 父工程
     <parent>
        <groupId>com.peng</groupId>
        <artifactId>Spring-Cloud-Bill</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>Eureka-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>Eureka-service</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        #设定Spring-Cloud 版本
        <spring-cloud-version>Greenwich.SR1</spring-cloud-version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
 ```
 #### 2.2.2 启动类 Application 设定
 ```
 @SpringBootApplication
@EnableEurekaServer
public class EurekaServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServiceApplication.class, args);
    }

}

 ```
 ### 2.3 搭建服务者
 #### 2.3.1 pom 依赖关系
|  SpringBoot     |
| :--- |
|   mysql    |
|   tk-Mybatis    |
|    thymeleaf   |
|  pagehelper |
| eureka |
| rabbitmq |
| spring-config |

 ```
  <parent>
        <groupId>com.peng</groupId>
        <artifactId>Spring-Cloud-Bill</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <artifactId>user-service</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>user-service</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <pagehelper.spring.version>1.2.3</pagehelper.spring.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.46</version>
        </dependency>
        <!-- 通用mapper -->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>2.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.20</version>
        </dependency>
        <!-- 分页插件-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>${pagehelper.spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-bus</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
 ```
 #### 2.3.2 生产者 配置文件
 ```
 spring:
  application:
  # 微服务的名称 使用RPC 调用时候 可以使用此名称
    name: billservice
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/bill-manager?useSSL=false
    username: root
    password: 123456
  thymeleaf:
    cache: false
#mybatis
mybatis:
  type-aliases-package: com.peng.pojo
  mapper-locations: classpath:/mappers/*.xml

# DataSource Config
#spring:
#  datasource:
#    driver-class-name: org.h2.Driver
#    schema: classpath:db/schema-h2.sql
#    data: classpath:db/data-h2.sql
#    url: jdbc:h2:mem:test
#    username: root
#    password: test
#  # Logger Config
#logging:
#  level:
#    com.lxs.quickstart: debug
server:
  port: ${port:9090}
eureka:
  client:
    service-url:
      defaultZone: HTTP://127.0.0.1:6060/eureka
  instance:
  # 更倾向使用ip地址，而不是host名
    prefer-ip-address: true
  # ip地址
    ip-address: 127.0.0.1
  # 续约间隔，默认30秒
    lease-renewal-interval-in-seconds: 5
  # 服务失效时间，默认90秒
    lease-expiration-duration-in-seconds: 5
ribbon:
  eureka:
    enabled: true
 ```
 #### 2.3. 创建服务接口
 1. 生产者的服务接口就使用平时的 Rest 风格接口 ，只需注意 Controller 返回类型 为 ``Json```  
 2. 此时需要使用 ```RestController``` 注解表明返回值为Json同时方法使用 的```RequestMapper```或者是```GetMapping```都需要同调用者的保持一致。
 3. 对于需要传参的参数 需要使用```@RequestBody```注解
 > 生产者代码

```
@RestController
@RequestMapping("/service")
public class UserProvider {

    @Resource
    private TypeService typeService;

    @Resource
    private BillService billService;


    @Resource
    private BillDTOReq billDTOReq;
    @Resource
    private BillDTORsp billDTORsp;

    @RequestMapping("/list-page")
    public BillDTORsp listPage( @RequestBody BillDTOReq billDTOReq) {
        System.out.println("---->>>>>>>>>>>>>>>>>>> bill-service/list-pages ");

        List<BillType> types = typeService.list();
        PageInfo<Bill> pageInfo = billService.listPage(billDTOReq.getBill(), billDTOReq.getPage(), billDTOReq.getSize());
        billDTORsp.setBillTypes(types);
        billDTORsp.setPageInfo(pageInfo);
        return billDTORsp;
    }
    @RequestMapping(value = "/test")
    public String test(@RequestBody String s){
        System.out.println("---------------------------------------------------");
        return s;
    }
}
```

### 2.4 消费者搭建
####  2.4.1 消费者 pom 依赖关系
```
   <parent>
        <groupId>com.peng</groupId>
        <artifactId>Spring-Cloud-Bill</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>customer-service</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>com.github.spt-oss</groupId>
            <artifactId>spring-boot-starter-test-plus</artifactId>
            <version>0.5.0</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>5.1.2</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```

#### 2.4.2 消费者配置文件

```
server:
 port: ${port:7070}
spring:
 application:
  name: consumer-demo
eureka:
 client:
  service-url:
   defaultZone: ${defaultZone:http://127.0.0.1:6060/eureka}
#   defaultZone: http://127.0.0.1:6060/eureka

  registry-fetch-interval-seconds: 10
bill-service:
 ribbon:
  NFLoadBalanecerRuleClassName: com.netflix.loadbalancer.RandomRule
```

#### 2.4.3 消费者工程搭建

> Application 启动类
```
@SpringBootApplication
@EnableDiscoveryClient // 开启Eureka客户端
@EnableFeignClients
public class CustomerApplication {
    public static void main(String[] args) {
        SpringApplication.run(CustomerApplication.class,args);
    }
}
```
> Service 接口
```
@FeignClient(value = "billservice",configuration = FeginConfig.class,fallback = BillServiceFallBack.class)
public interface BillService {
    @RequestMapping(value = "/service/test")
    public String test(@RequestBody String s);

    @RequestMapping(value = "/service/list-page")
    public BillDTORsp listPage(@RequestBody BillDTOReq billDTOReq);
```
> Config 配置
```
 @Configuration
public class FeginConfig {
    @Bean
    public Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }


}
```

> FallBack 类
```
public class BillServiceFallBack implements BillService{


    @Override
    public BillDTORsp listPage(BillDTOReq billDTOReq) {
        return "暂时无法访问 ，列表界面";
    }
    @Override
    public String test(String s) {
        return  "网络拥挤 服务降级";
    }
}
```