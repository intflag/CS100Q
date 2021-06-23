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
### Ribbon
- 首先 Ribbon 会从 Eureka Client 里获取到对应的服务注册表，也就知道了所有的服务都部署在了哪些机器上，在监听哪些端口号；
- 然后 Ribbon 就可以使用默认的 Round Robin 轮询算法算法，从中选择一台机器；
- Feign 就会针对这台机器，构造并发起请求；

![](http://images.intflag.com/springcloud003.jpg)

## 服务治理
### Hystrix
- 包裹请求：使用 HystrixCommand（或HystrixObservableCommand）包裹对依赖的调用逻辑，每个命令在独立线程中执行。这使用到了设计模式中的“命令模式”；
- 跳闸机制：当某服务的错误率超过一定阈值时，Hystrix可以自动或者手动跳闸，停止请求该服务一段时间；
- 资源隔离：Hystrix 为每个依赖都维护了一个小型的线程池（或者信号量）。如果该线程池已满，发往该依赖的请求就被立即拒绝，而不是排队等候，从而加速失败判定；
- 监控：Hystrix可以近乎实时地监控运行指标和配置的变化，例如成功、失败、超时、以及被拒绝的请求等；
- 回退机制：当请求失败、超时、被拒绝，或当断路器打开时，执行回退逻辑。回退逻辑可由开发人员自行提供，例如返回一个缺省值；
- 自我修复：断路器打开一段时间后，会自动进入“半开”状态。断路器打开、关闭、半开的逻辑转换；

![](http://images.intflag.com/springcloud004.jpg)

## 服务网关
### Zuul
- 身份认证与安全：识别每个资源的验证要求，并拒绝那些与要求不符的请求；
- 审查与监控：在边缘位置追踪有意义的数据和统计结果，从而为我们带来精确的生产视图；
- 动态路由：动态地将请求路由到不同的后端集群；
- 压力测试：逐渐增加指向集群的流量，以了解性能；
- 负载分配：为每一种负载类型分配对应容量，并弃用超出限定值的请求；
- 静态响应处理：在边缘位置直接建立部分响应，从而避免其转发到内部集群；
- 多区域弹性：跨越 AWS Region 进行请求路由，旨在实现 ELB（Elastic Load Balancing）使用的多样化；以及让系统的边缘更贴近系统的使用者。

![](http://images.intflag.com/springcloud005.jpg)

## 总结
- Eureka：各个服务启动时，Eureka Client 都会将服务注册到 Eureka Server，并且 Eureka Client 还可以反过来从 Eureka Server 拉取注册表，从而知道其他服务在哪里；
- Ribbon：服务间发起请求的时候，基于 Ribbon 做负载均衡，从一个服务的多台机器中选择一台；
- Feign：基于 Feign 的动态代理机制，根据注解和选择的机器，拼接请求 URL 地址，发起请求；
- Hystrix：发起请求是通过 Hystrix 的线程池来走的，不同的服务走不同的线程池，实现了不同服务调用的隔离，避免了服务雪崩的问题；
- Zuul：如果前端、移动端要调用后端系统，统一从 Zuul 网关进入，由 Zuul 网关转发请求给对应的服务；

![](http://images.intflag.com/springcloud006.jpg)