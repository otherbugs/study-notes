# Spring Cloud 学习笔记

## 一、微服务

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

- 仓库已经废弃了spring-cloud-starter-eureka-server，推荐使用spring-cloud-starter-netflix-eureka-server。

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

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```



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