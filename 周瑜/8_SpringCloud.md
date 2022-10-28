# 8_SpringCloud

## 什么是微服务架构？

- 微服务架构风格是一种使用**一套小服务来开发单个应用的方式途径，每个服务运行在自己的进程中**，并使用轻量级机制通信，通常是HTTP API，这些服务基于业务构建，并能够通过自动化部署机制来独立部署，这些服务使用不同的编程语言实现，以及不同数据存储技术，并保持最低限度的集中式管理。 去中⼼化，服务⾃治。
- 解决单体架构：扩展性差，可靠性不高，维护成本高，耦合度高问题。

## 微服务的优缺点

- 优点：
  - 服务简单，便于学习和上手，相对容易维护
  - 独立部署，灵活扩展
  - 技术栈丰富，不同模块之间可以使用不同的技术
- 缺点
  - 运维成本太高----需要过多的服务器，颗粒细化，维护成本高
  - 接口可能不匹配----session一致性需要去保持
  - 代码可能重复
  - 架构复杂度提高—前后端分离，Nginx配置以及多服务的部署，运维复杂性增加。

## SpringCloud

- Spring Cloud是微服务的开发框架，是**一系列框架的集合**。使用SpringCloud里面这些框架**实现微服务操作**。
- 使用SpringCloud， 需要**依赖技术SpringBoot**
- 它利用Spring Boot的开发便利性简化了分布式系统基础设施的开发，如服务发现、服务注册、配置中心、消息总线、负载均衡、 熔断器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。

## Spring Cloud和Spring Boot关系

- Spring Boot 是 Spring 的一套快速配置脚手架，可以基于Spring Boot 快速开发单个微服务，Spring Cloud是一个基于Spring Boot实现的开发工具；
- Spring Boot专注于快速、方便集成的单个微服务个体，Spring Cloud关注全局的服务治理框架； 
- Spring Boot使用了默认大于配置的理念，很多集成方案已经帮你选择好了，能不配置就不配置，Spring Cloud很大的一部分是基于Spring Boot来实现，必须基于Spring Boot开发。可以单独使用Spring Boot开发项目，但是Spring Cloud离不开 Spring Boot。

## 微服务的基础服务组件

- 服务发现——Netflix Eureka （Nacos）
- 服务调用——Netflix Feign
- 熔断器——Netflix Hystrix 
- 服务网关——Spring Cloud GateWay 
- 分布式配置——Spring Cloud Config  （Nacos）
- 消息总线 —— Spring Cloud Bus （Nacos）

## Spring Cloud具体调用

Spring Cloud 在接口调用上，大致会经过如下几个组件配合：

- 接口化请求调用：调用端 指定调用服务名称，定义调用的方法路径--->
- Nacos服务发现，找到需要调用的服务--->
- Feign服务调用--->
- Hystrix熔断器：需要调用的服务能不能调到（服务器宕机），调不到就熔断--->
- Ribbon负载均衡：将请求平均分担到集群的服务器中--->
- Http Client(apache http components 或者 Okhttp)：真正执行方法

## Nacos

![image-20221027164248780](Pic/image-20221027164248780.png)

- Nacos 是阿里巴巴推出来的一个新开源项目，是一个更易于构建云原生应用的动态**服务发现**、配置管理和服务管理平台。
- 服务发现功能：
  - 注册中心：
    - 注册中心使用Nacos
    - 注册中心是用来集中管理微服务，实现服务的注册，发现，检查等功能；
    - 实现不同的微服务模块之间调用，把这些模块在注册中心进行注册，注册之后，实现互相调用。
  - 注册中心和服务发现：服务 A 与服务 B 注册进注册中心 C，形成服务注册表（表里登记了服务 A 和服务 B 的地址等相关信息）。当 A 服务想要访问 B 服务时，可以通过注册中心 C 的**服务发现机制**，获取服务注册表进而找到服务 B；
  - 使用：
    - 启动类添加注解@EnableDiscoveryClient，该服务就注册到注册中心
- 分布式配置：使用配置中心实现。
  - 配置中心：做配置文件的统一管理
  - 以前，多个服务各自用自己的配置文件application.properties；现在，多个服务使用配置中心的一个配置文件application.properties。方便统一管理配置文件。
  - 配置文件加载顺序：bootstrap-->application(可以使用配置中心)-->application-dev（可以使用配置中心）

## Feign

- SpringCloud里面用来**服务调用**。Feign是Netflix开发的声明式、模板化的HTTP客户端， Feign可以帮助我们更快捷、优雅地调用HTTP API。
- 使用Spring Cloud feign，只需要创建一个接口并用注解方式配置它，即可完成服务提供方的接口绑定。
- 使用：前提，服务已经注册过。
  - 在**调用端**的启动类添加注解@EnableFeignClients
  - 在调用端 创建interface，使用注解指定调用服务名称，定义调用的方法路径
  - 调用

```java
//指定从哪个服务中调用功能 ，名称与被调用的服务名保持一致。
@FeignClient("service-vod")
@Component
public interface VodClient {
  //定义调用的方法路径
    @DeleteMapping(value = "/eduvod/vod/video/{videoId}")
  	//@PathVariabie注解一定要指定参数名称
    public R removeVideo(@PathVariable("videoId") String videoId);
}

//调用
vodClient.removeVideo(videoSourceId);
```

## Hystrix

- Hystrix 是一个供分布式系统使用，提供延迟(延长请求超时限制)和容错(服务器宕机，熔断机制触发，其他请求也不能请求该服务器)功能，保证复杂的分布系统在面临不可避免的失败时，仍能有其弹性。

- 使用：

  - 创建interface对应实现类，在实现类实现方法，出错了输出内容

  ```java
  @Component
  public class VodFileDegradeFeignClient implements VodClient {
      @Override
      public R removeVideo(String videoId) {
          return R.error().message("time out");
      }
      @Override
      public R removeVideoList(List videoIdList) {
          return R.error().message("time out");
      }
  }
  ```

  - 在interface上面添加注解和属性

  ```java
  @FeignClient(name = "service-vod", fallback = VodFileDegradeFeignClient.class)
  ```

## GateWay

- 网关：在客户端和服务端中间一面墙，可以请求转发，负载均衡，权限控制等。
- Nginx与网关都是可以实现对接口的拦截，负载均衡、反向代理、请求过滤等。
- 网关也是服务，要到注册中心注册；通过网关来访问其他服务，客户端访问网关的端口号
