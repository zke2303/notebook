# Spring-Cloud

## OpenFeign

Spring Cloud OpenFeign是一种基于Spring Cloud的声明式**REST客户端**，它**简化了与HTTP服务交互的过程**。它将REST客户端的定义转化为Java接口，并且可以通过注解的方式来声明请求参数、请求方式、请求头等信息，从而使得客户端的使用更加方便和简洁。同时，它还提供了负载均衡和服务发现等功能，可以与Eureka、Consul等注册中心集成使用。Spring Cloud OpenFeign能够提高应用程序的可靠性、可扩展性和可维护性，是构建微服务架构的重要工具之一。

#### 快速入门

1. 引入依赖，OpenFeign和LoadBalancer, LoadBalancer是用来负载均衡的

```xml
<!--openFeign-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--负载均衡器-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

2. 编写FeignClient

   ```java
   @FeignClient("item-service") // 服务名， item-service是 http://localhost:8081
   public interface ItemClient {
   
   	@GetMapping("/items") // 基于RestFul,告知这个是Get请求，请求路径是items，请求参数是ids, 最终的请求是
       					  // Get http://localhost:8081/items?ids=
   	List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
   }
   ```

3. 开启Feign功能， 在启动类中添加`@EnableFeignClients`注解

   ```java
   @EnableFeignClients(basePackages = "com.hmall.api.client", defaultConfiguration = DefaultFeignConfig.class) // 开启 openfeign 功能
   ```

4. 使用FeignClient，实现远程调用

   ```java
   // ItemClient.queryItemByIds(List.of(1, 2, 3)); 发送 Get http://localhost:8081/items?ids=1,2,3请求
   List<ItemDTO> items = ItemClient.queryItemByIds(List.of(1, 2, 3));
   ```



#### 连接池

Feign底层发起http请求，依赖于其它的框架。其底层支持的http客户端实现包括：

- HttpURLConnection：默认实现，不支持连接池
- Apache HttpClient ：支持连接池
- OKHttp：支持连接池

因此我们通常会使用带有连接池的客户端来代替默认的HttpURLConnection。比如，我们使用OK Http.

OK Http的使用：

1. 引入依赖

   ```xml
   <!--OK http 的依赖 -->
   <dependency>
     <groupId>io.github.openfeign</groupId>
     <artifactId>feign-okhttp</artifactId>
   </dependency>
   ```

2. 开启openfeign的连接池

```yml
feign:
  okhttp:
    enabled: true # 开启OKHttp功能
```



#### 日志

OpenFeign只会在FeignClient所在包的日志级别为**DEBUG**时（这个是日志级别是包的日志级别），才会输出日志。openfeign自己的日志级别有4级：

- **NONE**：不记录任何日志信息，这是默认值。
- **BASIC**：仅记录请求的方法，URL以及响应状态码和执行时间
- **HEADERS**：在BASIC的基础上，额外记录了请求和响应的头信息
- **FULL**：记录所有请求和响应的明细，包括头信息、请求体、元数据。

Feign默认的日志级别就是NONE，所以默认我们看不到请求日志。



1. 通过一个Logger.Level的Bean就可以定义Openfeign的日志级别

```java
public class DefaultFeignConfig {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.FULL;
    }
}
```

2. 接下来，要让日志级别生效，还需要配置这个类。有两种方式：

   - **局部**生效：在某个`FeignClient`中配置，只对当前`FeignClient`生效

   ```Java
   @FeignClient(value = "item-service", configuration = DefaultFeignConfig.class)
   ```

   - **全局**生效：在`@EnableFeignClients`中配置，针对所有`FeignClient`生效。

   ```Java
   @EnableFeignClients(defaultConfiguration = DefaultFeignConfig.class)
   ```

一般都是创建一个config包，然后在这个包里面创建一个Logger.Leverl的Bean，注意，不需要加入其他的注解，只需要一个@Bean就可以了





## 网关

什么是网关？

顾明思议，网关就是**网**络的**关**口。数据在网络间传输，从一个网络传输到另一网络时就需要经过网关来做数据的**路由**和转发以及数据安全的校验。

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGIwN2EzOTMzYzBjODhjMjFlMzc2ZTNmNDM4Y2M4MWJfMjRMSktCNXJkeTNQZ3VNZFRsN1JnR3VoZDM4UjkweE9fVG9rZW46TU9tZWJQTjhob1RDVmh4TjF0VGNicVVobmdoXzE3NTY3MTYyMDg6MTc1NjcxOTgwOF9WNA)

现在，微服务网关就起到同样的作用。前端请求不能直接访问微服务，而是要请求网关：

- 网关可以做安全控制，也就是登录身份校验，校验通过才放行
- 通过认证后，网关再根据请求判断应该访问哪个微服务，将请求转发过去

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=YzRjNDg3OWFmYTI3NjNjMDdlMmU0ZDNjMDkxNWYzNDlfWmlmYlJXekpTYzVtWnBMUnhmejFwTnFDZjFsRVZnamlfVG9rZW46WFdmaWJuMUpKb0loM0x4V21SdGNneHVsbm9lXzE3NTY3MTY1MzY6MTc1NjcyMDEzNl9WNA)

在SpringCloud当中，提供了两种网关实现方案：

- Netflix Zuul：早期实现，目前已经淘汰
- SpringCloudGateway：基于Spring的WebFlux技术，完全支持响应式编程，吞吐能力更强

现在的主流就是**SpringCloudGateway**

#### 快速入门

网关本身也是一个独立的微服务，因此也需要创建一个模块开发功能。大概步骤如下：

- 创建网关微服务
- 引入SpringCloudGateway、NacosDiscovery依赖
- 编写启动类
- 配置网关路由



1. 引入相关依赖

   ```xml
   <!--网关-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-gateway</artifactId>
   </dependency>
   <!--nacos discovery 服务发现-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   <!--负载均衡-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-loadbalancer</artifactId>
   </dependency>
   ```

   

2. 在网关微服务中，创建一个启动类

3. 配置路由

   接下来，在`hm-gateway`模块的`resources`目录新建一个`application.yaml`文件

   ```yml
   server:
     port: 8080
   spring:
     application:
       name: gateway
     cloud:
       nacos:
         server-addr: 127.0.0.1:8848
       gateway: # 网关
         routes: # 路由配置
           - id: item # 路由规则id，自定义，唯一
             uri: lb://item-service # 路由的目标服务，lb代表负载均衡，会从注册中心拉取服务列表
             predicates: # 路由断言，判断当前请求是否符合当前规则，符合则路由到目标服务
               - Path=/items/**,/search/** # 这里是以请求路径作为判断规则
           - id: cart
             uri: lb://cart-service
             predicates:
               - Path=/carts/**
           - id: user
             uri: lb://user-service
             predicates:
               - Path=/users/**,/addresses/**
           - id: trade
             uri: lb://trade-service
             predicates:
               - Path=/orders/**
           - id: pay
             uri: lb://pay-service
             predicates:
               - Path=/pay-orders/**
   
   ```

   

配置完成后，就可以把请求发送到网关，然后网关路由到相关微服务中

![image-20250901170755531](/home/zhang/.config/Typora/typora-user-images/image-20250901170755531.png)

上图所示，8080端口是网关的端口，成功访问到了items-service微服务



#### 路由属性

网关路由对于的Java类型是RouteDefinition, 其中常见的属性有：

- `id`：路由的唯一标示
- `predicates`：路由断言，其实就是匹配条件
- `filters`：路由过滤条件，对请求或响应做特殊处理
- `uri`：路由目标地址，`lb://`代表负载均衡，从注册中心获取目标微服务的实例列表，并且负载均衡选择一个访问。



SpringCloudGateway提供了12中路由断言：

| **名称**   | **说明**                       | **示例**                                                     |
| :--------- | :----------------------------- | :----------------------------------------------------------- |
| After      | 是某个时间点后的请求           | - After=2037-01-20T17:42:47.789-07:00[America/Denver]        |
| Before     | 是某个时间点之前的请求         | - Before=2031-04-13T15:14:47.433+08:00[Asia/Shanghai]        |
| Between    | 是某两个时间点之前的请求       | - Between=2037-01-20T17:42:47.789-07:00[America/Denver], 2037-01-21T17:42:47.789-07:00[America/Denver] |
| Cookie     | 请求必须包含某些cookie         | - Cookie=chocolate, ch.p                                     |
| Header     | 请求必须包含某些header         | - Header=X-Request-Id, \d+                                   |
| Host       | 请求必须是访问某个host（域名） | - Host=**.somehost.org,**.anotherhost.org                    |
| Method     | 请求方式必须是指定方式         | - Method=GET,POST                                            |
| Path       | 请求路径必须符合指定规则       | - Path=/red/{segment},/blue/**                               |
| Query      | 请求参数必须包含指定参数       | - Query=name, Jack或者- Query=name                           |
| RemoteAddr | 请求者的ip必须是指定范围       | - RemoteAddr=192.168.1.1/24                                  |
| weight     | 权重处理                       | - Weight=group1, 2 |
| XForwarded Remote Addr| 基于请求的来源IP做判断| - XForwardedRemoteAddr=192.168.1.1/24 |



#### 路由过滤器

网关中提供了33中路由过滤器，每种过滤器都有独特的作用：





#### 网关过滤器

![image-20250901202407864](/home/zhang/.config/Typora/typora-user-images/image-20250901202407864.png)

网关过滤器有两种，分别是：

- GatewayFilter: 路由过滤器，作用用任意指定的路由；默认不生效，要配置到路由后生效
- GlobalFilter:全局过滤器，作用范围是所有路由；声明后自动生效

#### 自定义过滤器

无论是`GatewayFilter`还是`GlobalFilter`都支持自定义，只不过**编码**方式、**使用**方式略有差别。

##### 自定义GlobalFilter：

自定义GlobalFilter简单很多，直接实现GlobalFilter即可，而且也无法设置动态参数：

```java
public class MyGlobalFilter implements GlobalFilter {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        return chain.filter(exchange);
    }
}
```

ServerWebExchange：请求上下文，包含整个过滤器链内共享数据

GatewayFilterChain：过滤器链，当前过滤器执行完毕后。要调用过滤器链中的下一个过滤器

需要把exchange通过过滤器链传递过下一个过滤器，不然的话，请求上下文无法在过滤器中共享。还有注意的一点是，过滤器的优先级问题，NettyRoutingFilter是一个把http请求转换到微服务的filter，因此这个过滤器的优先级必须在最后，其他过滤器执行完毕之和在经过这个Filter，Filter必须能够自定义优先级。那么怎么才能设定Filter的优先级呢？其实很简单，只需要实现`Ordered`接口即可，重写`Ordered`中的`getOrder()`方法。 这`Ordered`接口中，数字越低，优先级越高，最高的优先级是的`Long`类型的最小值，最低优先级是`Long`类型的最大值。

`NettyRoutingFilter`的默认优先级就是`Long`的最大值，也即是最低优先级。

##### 自定义GatewayFilter

自定义`GatewayFilter`不是直接实现`GatewayFilter`，而是实现`AbstractGatewayFilterFactory`。

简单实现：

1. 编写一个类继承AbstractGatewayFilterFactory， 并实现`apply`方法

```java
@Component
public class PrintAnyGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {
    @Override
    public GatewayFilter apply(Object config) {
        return new GatewayFilter() {
            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                // 获取请求
                ServerHttpRequest request = exchange.getRequest();
                // 编写过滤器逻辑
                System.out.println("过滤器执行了");
                // 放行
                return chain.filter(exchange);
            }
        };
    }
}
```

> **注意**：该类的名称一定要以`GatewayFilterFactory`为后缀！

2. 编写application.yml配置default-filter属性：

   ```yml
   spring:
     cloud:
       gateway:
         default-filters:
               - PrintAny # 此处直接以自定义的GatewayFilterFactory类名称前缀类声明过滤器, 所以定义的方法必须以GatewayFilterFactory为后缀
   ```

   

不过`GatewayFilter`无法指定`Ordered`， 所有一般都是new一个`OrederedGatewayFilter`对象。

``` java
@Component
public class PrintAnyGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {
    @Override
    public GatewayFilter apply(Object config) {
        return new OrderedGatewayFilter(
                new GatewayFilter() {
                    @Override
                    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
                        HttpHeaders headers = exchange.getRequest().getHeaders();
                        System.out.println("GatewayFilter....");
                        System.out.println("headers: " + headers);
                        return chain.filter(exchange);
                    }
                }, 1
        );
    }
}
```

