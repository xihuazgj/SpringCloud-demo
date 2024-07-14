# Springcloud相关学习笔记



Springboot + Mybatis-plus + docker + mysql + nginx + Springcloud + nacos（统一集群管理） + Openfeign（微服务之间的调用，使用它的拦截器以便在微服务之间传递信息）+sentinel(服务监控，服务限流，服务隔离，服务熔断)+seata(分布式事务，两种模式：XA和AT模式，XA遵循的是强一致性，服务之间会等待每个事务执行完毕才能提交，失败则一起回滚，消耗时间；AT模式是最终一致性，会经过两个阶段，第一阶段根据事务需求生成快照（undo log），第二阶段根据事务执行情况，若成功，则删除快照，结束，失败则通过快照进行数据恢复，回滚。中间会出现一段时间的数据不一致)







事务（Transaction）是一个核心概念。事务的四个特性，通常被称为ACID特性，是确保数据库操作可靠性和一致性的基石。这四个特性分别是：

1. **原子性（Atomicity）**：
   原子性指的是事务中的操作要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误会被回滚（Rollback）到事务开始前的状态，就像这个事务从未执行过一样。
2. **一致性（Consistency）**：
   一致性是指事务必须使数据库从一个一致性状态变换到另一个一致性状态。事务的结束不会破坏数据库的完整性约束，并且所有相关的数据规则都必须得到满足。
3. **隔离性（Isolation）**：
   隔离性是指数据库系统提供一定的隔离级别，使得事务在不受外部并发操作影响的“独立”环境中运行。这意味着事务处理过程中的中间状态对外部是不可见的，或者说，事务不可分割地执行，不会被其他事务干扰。不同的隔离级别可以防止不同的并发问题，如脏读、不可重复读和幻读。
4. **持久性（Durability）**：
   持久性是指一旦事务被提交，它对数据库的修改就是永久性的，即使系统发生故障也不会丢失。即使发生系统崩溃，数据库也能通过日志文件等机制恢复到最后一次提交的状态。

## 悲观锁与乐观锁

### 一、定义与思想

- **悲观锁（Pessimistic Locking）**：悲观锁假设在事务执行期间，其他事务很可能会对共享资源进行修改，因此默认情况下会将资源锁定，以阻止其他事务的访问。它适用于对数据访问频率高、竞争激烈的情况。
- **乐观锁（Optimistic Locking）**：乐观锁则假设在事务执行期间，不会有其他事务对共享资源进行修改，因此不会对资源进行锁定。其核心思想是在事务读取数据时记录下当前的版本号或时间戳，然后在更新数据时检查这个版本号或时间戳是否发生变化。

### 二、实现方式

- 悲观锁
    - **数据库层面**：如MySQL中的排他锁（SELECT ... FOR UPDATE），可以锁定所选行，确保其他事务无法修改这些行直到锁被释放。
    - **代码层面**：如Java中的synchronized关键字，可以对代码块进行加锁，防止多个线程同时执行。
- 乐观锁
    - **版本号机制**：在数据中增加一个version字段，每次数据被修改时版本号加1。更新数据时检查版本号是否一致，一致则执行更新，不一致则放弃或重新尝试。
    - **CAS（Compare and Swap）机制**：涉及三个操作数，内存位置（V）、预期值（A）和新值（B）。如果内存位置的值等于预期值，则将其更新为新值，否则不操作。

### 三、应用场景

- **悲观锁**：适合用在写操作多、竞争激烈的场景中，如金融交易系统、库存管理系统等。这些系统对数据的一致性和完整性要求极高，需要通过加锁来确保数据的正确性。
- **乐观锁**：适合用在读操作多、写操作少且并发冲突概率较小的场景中，如博客系统、新闻网站等。这些系统对数据的实时性要求不高，可以通过乐观锁来提高系统的并发性能。

### 四、优缺点

- 悲观锁
    - **优点**：能够确保数据的一致性和完整性，避免数据冲突。
    - **缺点**：可能会降低系统的并发性能，因为其他事务需要等待锁被释放才能访问数据。此外，加锁和释放锁也需要消耗系统资源。
- 乐观锁
    - **优点**：不会阻塞其他事务的读取操作，提高了系统的并发性能。同时，由于不需要加锁和释放锁，也减少了系统资源的消耗。
    - **缺点**：在并发冲突概率较高的场景中，乐观锁可能会因为数据被反复修改而更新失败，导致CPU资源的浪费。此外，乐观锁的实现需要依赖于业务逻辑，增加了实现的复杂度。





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

