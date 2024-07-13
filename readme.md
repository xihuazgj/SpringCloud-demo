![Snipaste_2024-07-13_13-48-14.jpg](https://img.picui.cn/free/2024/07/13/66921497468ce.jpg)
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

### 1.配置UserInfo拦截器后，启动gateway，出现错误：

class path resource [org/springframework/web/servlet/config/annotation/WebMvcConfigurer.class] cannot be opened because it does not exist

类路径资源[orgspringframeworkwebservletconfigannotationwebmvcconfiguration .class]无法打开，因为它不存在



解决方案：

在common包中的configs目录下加上注解

```
@ConditionalOnClass(DispatcherServlet.class)
```

`@ConditionalOnClass(DispatcherServlet.class)` 是 Spring Framework 中的一个条件注解，它属于 Spring Boot 的自动配置（autoconfiguration）特性。这个注解用于条件化地注册一个 bean 或者配置类，只有当指定的类（在这个例子中是 `DispatcherServlet.class`）存在于类路径（classpath）上时，才会被注册或激活。

`DispatcherServlet` 是 Spring MVC 的核心，负责处理 HTTP 请求并将其转发到相应的控制器（Controller）。因此，`@ConditionalOnClass(DispatcherServlet.class)` 通常用于确保只有在 Spring MVC 可用（即 `DispatcherServlet` 类存在于类路径中）的情况下，相关的自动配置或 bean 才会被注册。

这种条件注解的使用场景包括但不限于：

1. **Spring Boot 自动配置**：Spring Boot 提供了大量的自动配置，这些配置会根据类路径中的类和已配置的属性自动配置应用。使用 `@ConditionalOnClass` 可以确保只有当特定类（如 `DispatcherServlet`）存在时，相关的自动配置才会被激活。

2. **条件化地注册 Bean**：在自定义的配置类中，你也可以使用 `@ConditionalOnClass` 来条件化地注册 Bean。例如，你可能想基于某个库的存在来注册特定的服务或组件。

3. **避免不必要的依赖**：如果你的应用不使用 Spring MVC，但你的项目依赖中包含了 Spring Boot 的 Web 起步依赖，那么通过 `@ConditionalOnClass(DispatcherServlet.class)`，你可以确保那些仅适用于 Spring MVC 的自动配置不会被激活，从而避免了不必要的依赖和潜在的冲突。

总之，`@ConditionalOnClass(DispatcherServlet.class)` 是一种强大的机制，用于根据类路径中的类存在性来条件化地激活或注册 Spring 应用中的配置或 bean。这有助于减少配置错误，提高应用的灵活性和可维护性。



### 2.如何在openfeign中传递用户

OpenFeign中提供了一个拦截器接口，所有由OpenFeign.发起的请求都会先调用拦截器处理请求



在微服务的api模块中配置一个类，用来拦截userInfo，用于在微服务之间传递

```
@Bean
public RequestInterceptor userInfoRequestInterceptor(){

    return new RequestInterceptor() {
        @Override
        public void apply(RequestTemplate requestTemplate) {
            Long userId = UserContext.getUser();
            if (userId != null){
            requestTemplate.header("user-info",userId.toString() );
            }
        }
    };
}
```

```
import com.hmall.api.config.DefaultFeignconfig;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.openfeign.EnableFeignClients;

@EnableFeignClients(basePackages = "com.hmall.api.client",defaultConfiguration = DefaultFeignconfig.class)
@MapperScan("com.hmall.pay.mapper")
@SpringBootApplication
public class PayApplication {
    public static void main(String[] args) {
        SpringApplication.run(PayApplication.class, args);
    }
}
```
注意要在接受userInfo的模块启动类
配置@EnableFeignClients(basePackages = "com.hmall.api.client",defaultConfiguration = DefaultFeignconfig.class)

<img align="left" width="300px" src="https://img.picui.cn/free/2024/07/13/669280db7ad25.jpg" />
<img align="right" width="300px" src="https://img.picui.cn/free/2024/07/13/669280db4bd7c.jpg" />
<img align="left" width="300px" src="https://img.picui.cn/free/2024/07/13/669280db6cf02.jpg" />
<img align="right" width="300px" src="https://img.picui.cn/free/2024/07/13/669280db7b239.jpg" />
<img align="center" width="300px" src="https://img.picui.cn/free/2024/07/13/669280dba6e7c.jpg" />
