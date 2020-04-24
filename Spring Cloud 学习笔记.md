# Spring Cloud 学习笔记

## 一、微服务

### Spring Boot Actuator 健康检查、审计、统计、监控

| Endpoint ID | Description                   |
| ----------- | ----------------------------- |
| health      | 应用的健康信息                |
| info        | 应用的基本信息                |
| mappings    | 显示所有的@RequestMapping路径 |
|             |                               |
|             |                               |
|             |                               |
|             |                               |
|             |                               |
|             |                               |
|             |                               |
|             |                               |
|             |                               |
|             |                               |



## 二、简介 Spring Cloud

### 2.1 简介

快速构建分布式系统的通用模式的工具集；

### 2.2 特点

- 约定由于配置；
- 适用于各种环境；
- 轻量级；
- 隐藏了组建的复杂性，并提供声明式、无 xml 的配置方式；

### 2.3 版本

大多数 Spring 项目采用 “主版本号.次版本号.增量版本号.里程碑版本号” 的姓氏命名；

增量版本号一般表示 Bug 修复；

**Spring Cloud 版本号**

采用 “英文单词 SRX(X为数字)” 的形式命名；

1. 英文单词叫 “release train”，例 Camden、Delston、Edgware 等都是伦敦地铁的名字，按字母顺序发行，主版本的演进；
2. SR 表示 “Service Release”，一般表示 Bug 修复；
3. 在 SR 发布之前，会先发布一个 Release 版本；例如在发布 Edgware SR1 之前会先发布 Edgware Release；

**Spring Cloud、Spring boot  版本兼容性**

1. Camden 版本基于 Spring Boot 1.4.x 构建；
2. Dalston 版本基于 Spring Boot 1.5.x 构建， 不兼容 Spring Boot 2.0.x；
3. Edgware 版本基于 Spring Boot 1.5.x 构建， 不兼容 Spring Boot 2.0.x；
4. Finchley 版本基于 Spring Boot 2.0.x 构建，不兼容 Spring Boot 1.x；



## 三、使用 Spring Cloud

本笔记 Spring Cloud 基于 Edgware RELEASE 版本；Spring Boot 基于 1.5.9.RELEASE;

- @SpringBootApplication 是一个组合注解，@Configuration、@EnableAutoConfiguration、@CompontScan；
- yml 格式有严格的缩紧；key 与 value 之间用 ：分隔，并且冒号后面有空格； 



## 四、Eureka 服务注册与发现

### 4.1 服务发现

**功能：**

- 服务注册表
- 服务注册与发现：微服务启动时，将自己的信息注册到服务发现组件上的过程；
- 服务检查

### 4.2 Eureka 简介

是一个基于 REST 的服务，包含 Server 和 Client 两个组件；

- Eureka Server 提供服务发现的能力，每个微服务启动时，都会向其注册自己的信息（IP、端口、服务名称等）；
- 微服务启动后，会周期性（默认30秒）向Eureka Server 发送心跳以续约自己的“租期”，如果 Eureka Server 一定时间内没有收到某服务的心跳，将注销该实例（默认90秒）；
- 默认情况下，Eureka Server 同时也是 Eureka Client。
- Eureka Client 会缓存服务注册表中的信息；

### 4.3 Eureka 使用

**1、加入依赖**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

- 仓库已经废弃了spring-cloud-starter-eureka-server，推荐使用spring-cloud-starter-netflix-eureka-server。不知道那个哪个版本开始的，c开头的还没变

**2、启动类添加注解 @EnableEurekaServer**

**3、配置文件**

```yaml
#本身是注册中心，但是也需要注册，向自己注册
eureka:
  client:
    #是否将自己注册到Eureka Server,自己不需要，true会报错
    register-with-eureka: false
    #是否从Eureka Server 获取注册信息，因为是单点Eureka Server，所以不需要
    fetch-registry: false
    #交互的地址，查询服务和注册服务都需要这个地址
    service-url:
      defaultZone: http://localhost:8761/eureka
```

### 4.4 将微服务注册到 Eureka Server

**1、添加依赖**

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**2、配置文件**

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    #将自己IP地址注册到 Eureka Server 上；
    prefer-ip-address: true
```

在 Spring cloud Edgware 之前，启动类上必须加启动注解；在 Edgware 之后，只需添加依赖就可以，即可自动注册；

### 4.5 Eureka Server 的高可用

Eureka Server 可通过运行多个实例并相互注册的方式实现高可用部署。实例会彼此增量的同步信息，从而确保所有节点数据一致；

### 4.6 用户认证

Eureka Server 是允许匿名访问的；实际项目中，可能必须经过用户认证才可以访问 Eureka Server；	

**1、添加依赖**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**2、配置文件**

```yaml
security:
	basic:
		enabled: true					#开启基于 HTTP Basic 的认证
	user:
		name: user						#配置登陆账号
		password: password123	#配置登陆密码
```

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://user:password123@localhost:8761/eureka/
```



## 五、Ribbon 客户端负载均衡

### 5.1 Ribbon 简介

Ribbon 默认提供了很多负载均衡算法；例如轮询、随机等；也可以洗定义负载均衡算法；

### 5.2 Ribbon 使用

#### 1、添加依赖

其实已经添加了spring-cloud-starter-eureka,就不需要添加ribbon的依赖了 ，因为Eureka包含了；

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

- 在使用 RestTemplate 的时候，如果 RestTemplate 上面有这个 @LoadBalanced 注解，那么这个 RestTemplate 调用的远程地址，会走负载均衡；



## 六、使用 Feign 实现声明式 REST 调用

### 6.1 Feign 简介

声明式、模板化的HTTP客户端；

### 6.2 Feign 使用

#### 6.2.1 添加依赖

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
```

#### 6.2.2 创建一个 Feign 接口，并添加 @FeignClient 注解；

**feign本身里面就包含有了ribbon**，就是这个注解 @FeignClient;

#### 6.2.3 修改启动类，添加注解 @EnableFeignClients

### 6.3 自定义 Feign 配置

Spring Cloud允许通过注解 @FeignClient 的 configuration属性自定义Feign的属性，自定义的比自带的 FeignClientsConfiguration 优先级要高；

```java
@FeignClient(name = "microservice-provider-user", configuration = FeignConfiguration.class)
public interface UserFeignClient{}
```

- 与Ribbon 配置自定义一样，自定义的FeignConfiguration类也不能包含在主应用程序上下文的@ComponentScan 扫描的包重叠，否则自定义的配置信息会被所有的@FeignClient共享；

### 6.4 Feign 的日志

- Feign 的 Logger.Level 只对 DEBUG 作出响应；

Logger.level的级别有

- NONE：不记录任何日志；
- BASIC：仅记录请求方法，URL、响应状态码以及执行时间；
- HEADERS：记录 BASIC 级别的基础上，记录请求和响应的 header；
- FULL：记录请求和响应的 header ，body 和 元数据；

#### 6.4.1 编写 Feign 的配置类

```java
@Configuration
public class FeignLogConfiguration {
  @Bean
  Logger.Level feignLoggerLevel() {
    return Logger.Level.BASIC;
  }
}
```

### 6.5 使用 Feign 构造多参数的请求

#### 6.5.1 GET 多参数

- 方法一 @RequestParam
  对每个参数前面都加上注解 @RequestParam(“”)；
- 方法二 Map

#### 6.5.2 POST 多参数

如果服务提供者的Controller的多参数前注解@RequestBody，那么请求的Feign 同理，多参数前面用该注解，例：`public post(@RequestBody User)`；



## 七、 Hystrix 实现容错处理

### 7.1 实现容错手段

#### 7.1.1 雪崩效应

把“<u>基础服务故障</u>”导致“<u>级联故障</u>”的现象称为**雪崩效应**；

#### 7.1.2 如何容错

- 为网络请求设置超时时间；
- 使用断路器模式；
  - 对容易导致错误的操作的代理；这种代理能够统计在一段时间内调用失败的次数，并决定正常请求，还是直接返回；
  - 如果在一段时间内检测到许多类似的错误（例如超时）就会强迫对该服务的调用快速失败；
  - 如果发现依赖服务正常，就会回复请求该服务；

### 7.2 使用 Hystrix 实现容错

Hystrix 是一个实现了超时机制和断路器模式的工具类库；

#### 7.2.1 Hystrix 简介

一个延迟和容错库；通过以下几点实现延迟和容错；

- **包裹请求**：使用到了命令模式；
- **跳闸机制**：当某服务的<u>错误率超过一定阈值时，Hystrix 可以自动或手动跳闸</u>停止请求服务一段时间；
- **资源隔离**：Hystrix 为每个依赖都维护了一个小型的线程池，<u>若线程池满了，发往该依赖的请求就被立刻拒绝</u>，而不是等待；
- **监控**
- 回退机制：断路器打开时，执行回退逻辑，可有开发人员自行提供，例如返回一个缺省值；
- 自我修复：断路器打开一段时间后，会自动进入“半开”状态，可允许**一个**请求访问该依赖的服务；成功就关闭断路器，否则保持打开；

#### 7.2.2 整合 Hystrix

1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```

2、启动类添加启动注解

启动类上添加**@EnableCircuitBreaker**或**@EnableHystrix**

 3、让某方法具有容错能力

注解  **@HystrixCommand(fallbackMethod** = "fallbackMethod "**)**

```java
 @HystrixCommand(fallbackMethod = "findByIdFallback")
  @GetMapping("/user/{id}")
  public User findById(@PathVariable Long id) {
    return this.restTemplate.getForObject("http://microservice-provider-user/" + id, User.class);
  }

  public User findByIdFallback(Long id) {
    User user = new User();
    user.setId(-1L);
    user.setName("默认用户");
    return user;
  }
```

该回退方法与方法具有相同的参数和返回值类型；

**进入回退方法并不代表断路器打开；**因为失败率要达到阈值（5秒内20次）才关闭；

#### 7.2.3 线程隔离策略与传播上下文

两种 线程隔离（默认），信号量隔离；

一般来说，只有当调用负载非常高时（每秒数百次）才需要信号隔离；

#### 7.2.4 Feign 使用 Hystrix

Spring Cloud**默认已为Feign整合了Hystrix**

```java
@FeignClient(name = "microservice-provider-user", fallback = FeignClientFallback.class)
public interface UserFeignClient {
  @RequestMapping(value = "/{id}", method = RequestMethod.GET)
  public User findById(@PathVariable("id") Long id);

}

/**
 * 回退类FeignClientFallback需实现Feign Client接口
 */
@Component
class FeignClientFallback implements UserFeignClient {
  @Override
  public User findById(Long id) {
    User user = new User();
    user.setId(-1L);
    user.setUsername("默认用户");
    return user;
  }
}
```



**检查回退原因**

可以使用注解 @FeignClient 的 **fallbackFactory** 属性；

`@FeignClient(name = "microservice-provider-user", fallback = FeignClientFallback.class)`

需要实现FallbackFactory接口，并覆盖 create 方法

fallbackFactory 属性还有很多其他用途，让不同的异常返回不同的回退结果；



**Feign禁用Hystrix**

全局的禁用的话只需在application.yml中配置 feign.hystrix.enable=false即可；

部分的话就是编写配置类，在@FeignClient里引用该配置类即可；

### 7.3 Hystrix 的监控

只需添加依赖，就可使用/Hystrix.stream 断点获得Hystrix的监控信息；

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 7.4 Hystrix Dashboard 可视化监控

1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-dashboard</artifactId>
</dependency>
```

2、启动类上添加 **@EnableHystrixDashboard**

### 7.5 使用 Turbine 聚合监控数据

Turbine 是一个聚合 Hystrix 监控数据的工具，它可将所有相关的 /Hystrix.stream 端点数据集合到一个组合的 /turbine.steam

#### 7.5.1 Turbine 的使用

1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-turbine</artifactId>
</dependency>
```

2、启动类上添加 **@EnableTurbine**；

3、配置文件

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
turbine:
  appConfig: microservice-consumer-movie,microservice-consumer-movie-feign-hystrix-fallback-stream
  clusterNameExpression: "'default'"
```



#### 7.5.2 使用消息中间件收集数据 

先安装RabbitMQ

1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hystrix-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-stream-rabbit</artifactId>
</dependency>
```

3、配置文件

```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
	password: guest
```



*在Spring Cloud Camden SR4 中，依赖`spring-cloud-starter-turbine`不能与`spring-cloud-starter-turbine-stream`共存，否则会报异常；

## 八、 使用 Zuul 构建微服务网关

### 8.1 为什么使用微服务网关

若客户端直接与各个微服务通信，会有以下问题：

- 客户端会多次请求不同的微服务，增加客户端的复杂性；
- 存在跨域请求，在一定场景下处理相对复杂；
- **认证复杂**，每个服务需要独立认证；
- 难以重构；
- 某些微服务可能使用了防火墙/浏览器不友好的协议，直接访问会有一定的困难；

### 8.2 Zuul 简介

zuul 是 Netflix 开源的微服务网关，Zuul的核心是一系列的过滤器，这些过滤器完成以下功能：

- 身份认证安全
- 审查与监控
- 动态路由：动态地将请求路由到不同的后端集群；
- 压力测试：逐渐增加指向集群的流量，以了解性能；
- 负载分配：为每一种负载类型分配对应容量，并弃用超出限定值的请求；
- 静态响应处理
- 多区域弹性：跨域AWS Region 进行请求路由，旨在实现ELB使用的多样化，以及让系统的边缘更贴近系统的使用者；

### 8.3 编写 Zuul 微服务网关

1、添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-eureka</artifactId>
</dependency>
```

2、启动类添加注解 **@EnableZuulProxy**

声明一个Zuul代理，该代理使用Ribbon来定位用注册在Eureka Server中的微服务，同时，该代理还整合了Hystrix，从而实现容错，所有经过Zuul的请求都会在Hystrix命令执行；

**默认情况下，Zuul会代理所有注册到Eureka Server 的微服务；**路由规则如下：

http://ZUUL_HOST:ZUUL_PORT/微服务在Eureka 的 serviceId/**；

### 8.4 Zuul 的路由端点

当@EnableZuulProxy 与 Spring Boot Actuator配合使用时，Zuul会暴露一个路由管理端点/routes;

`spring-cloud-starter-zuul` 已经包含了 `spring-boot-starter-actuator`

### 8.5 路由配置详解

1. 自定义指定微服务的访问路径；

   ```yaml
   zuul: 
     routes: 
     	microservice-provider-user: /user/**
   ```

   

   将microservice-provider-user微服务被映射到 /user/**路径，其他微服务还是原来规则；

2. 忽略指定服务；
   可以使用 zuul.ignored-services 配置忽略的服务，多个符号分隔；

   ```yaml
   zuul: 
     ignored-services: microservice-user,microservice-movie
   ```

3. 忽略所有微服务，只路由指定微服务

   ```yaml
   zuul: 
     ignored-services: '*'
     routes: 
     	microservice-provider-user: /user/**
   ```

4. 同时指定微服务的serviceId和对应路径；

   ```yaml
   zuul: 
     route: 
     	service-id: microservice-provider-user
     	path: /user/**
   ```

   效果和1一样；

5. 同时指定path和url;

   ```yaml
   zuul: 
     routes: 
     	user-route:
     	  url: http://localhost:8000/
     	  path: /user/**
   ```

   使用这种方式配置的路由不会作为 HystrixCommand执行，同时不能使用 Ribbon 来负载均衡多个URL；例6可以解决问题；

6. 同时指定path 和 URL，并且不破坏Zuul 的Hystrix、Ribbon特性；

   ```yaml
   zuul: 
     routes: 
     	user-route:
     	  path: /user/**
     	  service-id: microservice-provider-user
   ribbon:
     eureka:
     	enable: false   # 为Ribbon禁用 Eureka
    mircoservice-provider-user:
      ribbon:
        listOfServers: localhost:8000,localhost:8001
   ```

7. 使用正则指定Zuul的路有规则
   借助 PatternServiceRouteMapper

   ```java
     @Bean
     public PatternServiceRouteMapper serviceRouteMapper() {//调用构造函数PatternServiceRouteMapper(String servicePattern,String routePattern)
         //servicePattern 指定微服务的正则
         //routePattern 指定路由正则
       return new PatternServiceRouteMapper("(?<name>^.+)-(?<version>v.+$)","${vesion}/${name}");
     }
   ```

8. 路由前缀
   示例1：

   ```yaml
   zuul: 
     prefix: /api
     strip-prefix: false
     routes: 
     	microservice-provider-user: /user/**
   ```

   访问/api/microservice-provider-user/1 会被转发到 microservice-provider-user 的 /api/1

   示例2：

   ```yaml
   zuul: 
     routes: 
     	microservice-provider-user: 
     	  path: /user/**
     	  strip-prefix: false
   ```

   ​	

9. 忽略某些路径
   可使用 ignoredPatterns，指定忽略的正则

   ```yaml
   zuul:
     ignoredPatterns: /**/admin/** #忽略所有包含/admin/的路径
     routes:
       microsevice-provider-user: /user/**
   ```

   

## 九、 Spring Cloud Config 统一配置

### 9.1 编写 config server

1. 添加依赖

   ```xml
   <dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-config-server</artifactId>
   </dependency>
   ```

2. 启动类添加注解 @EnableConfigServer

3. 配置文件

   ```yaml
   spring:
    cloud:
     config:
      server:
       git:
        uri: https://.....
        username: root
        password: 123456
   ```

### 9.2 编写 config client

1. 添加依赖

   ```xml
   <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
   <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

   

2. 配置文件：**bootstrap.yml**
   不是application.yml!!!

   ```yaml
   spring:
     application:
       name: microservice-foo    # 对应config server所获取的配置文件的{application}
     cloud:
       config:
         uri: http://localhost:8080/
         profile: dev            # profile对应config server所获取的配置文件中的{profile} 
         label: master           # 指定Git仓库的分支，对应config server所获取的配置文件的{label}
   ```

### 9.3 Config 与 Eureka 配合







# 其他知识点

### Netflix

是一家在线影片租赁提供商；

###  AWS

亚马逊云服务；

### 搭建多模块项目

例如微服务

1. 先 new 一个 maven 项目；
2. 删除 src 文件夹，并修改 pom 文件；
3. 然后在这个项目上 new module；
4. 修改子项目 pom



# demo顺序

## 第三章

**服务提供者**

microservice-simple-provider-user

**服务消费者**

microservice-simple-consumer-movie

## 第四章 Eureka

**编写Eureka Server**

microservice-discovery-eureka

**微服务注册到Eureka Server**

microservice-provider-user

**Eureka Server 的高可用**

microservice-discovery-eureka-ha

**为 Eureka server 添加用户认证**

microservice-discovery-eureka-authenticating

**Eureka 的元数据**

microservice-provider-user-my-metadata

microservice-consumer-movie-understanding-metadata

## 第五章 Ribbon

**为服务消费者整合Ribbon**

microservice-consumer-movie-ribbon

**使用Java代码自动配置**

mircoservice-consumer-movie-ribbon-customizing

**使用属性自定义Ribbon配置**

mircoservice-consumer-movie-ribbon-customizing-properties

**是 Ribbon 脱离 Eureka**

microservice-consumer-movie-without-eureka

## 第六章 Feign

**为消费者整合 Feign**

microservice-consumer-movie-feign

**自定义Feign配置**

microservice-consumer-movie-feign-customizing

**手动创建Feign**

microservice-provider-user-with-auth

microservice-consumer-movie-feign-manual

**Feign 的日志**

microservice-consumer-movie-feign-logging

**构造多参数的请求**

microservice-provider-user-multiple-params

microservice-consumer-movie-feign-multiple-params

## 第七章 Hystrix

**整合 Hystrix**

microservice-consumer-movie-ribbon-hystrix

**Feign 使用 Hystrix**

microservice-consumer-movie-feign-hystrix-fallback

**检查回退原因**

microservice-consumer-movie-feign-hystrix-fallback-factory

**监控信息**

microservice-consumer-movie-feign-hystrix-fallback-stream

**Hystrix Dashboard 监控可视化数据**

microservice-hystrix-dashboard

**Turbine 监控多个服务**

microservice-hystrix-turbine

**使用RabbitMQ消息中间件收集数据** 

microservice-consumer-movie-ribbon-hystrix-turbine-mq

microservice-hystrix-turbine-mq

## 第八章 Zuul

**编写网关**

microservice-gateway-zuul

## 第九章 Config

**编写 Config Server**

microservice-config-server

**编写 Config Client**

microservice-config-client

**编写/refresh 端点手动刷新配置**

microservice-config-client-refresh

**springcloud bus 自动刷新**

microservice-config-client-refresh-cloud-bus

**config 与 Eureka 结合**

microservice-config-server-eureka

microservice-config-client-eureka

**config 的用户认证**

microservice-config-server-authenticating

microservice-config-client-authenticating