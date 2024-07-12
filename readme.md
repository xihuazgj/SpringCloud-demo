1 .如何利用OpenFeign实现远程调用？

1. -引入OpenFeign和SpringCloudLoadBalancer依赖
2. -利用@EnableFeignClients:注解开启OpenFeign功能
3. -编写FeignClient

2.如何配置OpenFeign的连接池？

1. ·引入http客户端依赖，例如OKHttp、HttpClient
2. 配置yaml文件，打开OpenFeign连接池开关
3. OpenFeign使用的最佳实践方式是什么？
4. 由服务提供者编写独立module,将FeignClient及DTO抽取

3.如何配置0 penFeign输出日志的级别？

1. ·声明类型为Logger..Level的Bean
2. 在@FeignClient:或@EnableFeignClients注解上使用