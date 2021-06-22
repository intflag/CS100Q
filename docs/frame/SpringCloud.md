# Spring Cloud
### 参考资料
- [Spring Cloud底层原理](https://shishan100.gitee.io/docs/#/./docs/microservice/page1)

## 注册中心
### Eureka
- Eureka Client：负责将这个服务的信息注册到 Eureka Server 中；
- Eureka Server：注册中心，里面有一个注册表，保存了各个服务所在的机器和端口号；

![](http://images.intflag.com/springcloud001.jpg)

## 远程调用
### Feign
- 接口定义了 @FeignClient 注解，Feign 就会针对这个接口创建一个动态代理；
- 调用那个接口，本质就是会调用 Feign 创建的动态代理，这是核心中的核心；
- Feign 的动态代理会根据你在接口上的 @RequestMapping 等注解，来动态构造出你要请求的服务的地址；
- 最后针对这个地址，发起请求、解析响应；

![](http://images.intflag.com/springcloud002.jpg)

## 负载均衡
## 限流 & 降级
## 网关