---
title: SpringBoot面试题
date: 2026-06-07
type: page
---

# SpringBoot面试题

> 共 19 道面试题

## 1. Spring Boot 工程启动以后，我希望将数据库中已有的固定内容，打入到 Redis 缓存中，请问如何处理？

🟢 **简单** | 标签：Spring Boot, MySQL, Redis

Spring Boot 启动后预热缓存，推荐通过 `CommandLineRunner` 或 `ApplicationRunner` 接口实现。这两个接口会在应用完全启动后、`main` 方法执行前自动运行，非常适合进行数据初始化。

具体实现方案如下：

1.  **实现接口**：创建一个 Bean 类，实现 `CommandLineRunner` 接口，并重写 `run` 方法。
2.  **注入依赖**：在该 Bean 中注入需要的数据源（如 JPA Repository、JdbcTemplate 等）和 Redis 操作类（如 `StringRedisTemplate`）。
3.  **加载数据**：在 `run` 方法中，查询数据库获取固定内容。
4.  **写入缓存**：遍历查询结果，使用 Redis 操作类将数据以合适的 Key-Value 结构（如 String、Hash）写入 Redis。建议设置合理的过期时间。
5.  **注意事项**：
    *   应处理可能发生的异常，避免因缓存加载失败影响应用正常启动。
    *   对于大型数据集，需考虑分批读写和内存占用。
    *   在分布式环境下，需确保此操作仅在单个实例中执行，或通过分布式锁协调，避免重复加载。

此方式利用了 Spring Boot 的生命周期钩子，实现简单、职责清晰，是面试中常见且被认可的解决方案。

---

## 2. Spring Boot 是否可以使用 XML 配置 ?

🟢 **简单** | 标签：Spring Boot, 后端

# Spring Boot 是否可以使用 XML 配置

**可以。** Spring Boot 支持使用 XML 配置，但推荐优先使用注解和 Java 配置类。

---

**引入 XML 配置的方式：**

通过 `@ImportResource` 注解加载 XML 文件：

```java
@SpringBootApplication
@ImportResource(locations = {"classpath:beans.xml"})
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

---

**适用场景：**

- **遗留系统迁移**：老项目已有大量 XML 配置，需要逐步迁移到 Spring Boot
- **第三方框架集成**：部分第三方组件要求使用 XML 方式配置 Bean
- **团队技术栈统一**：团队习惯使用 XML 配置风格

---

**Spring Boot 推荐的配置方式：**

| 优先级 | 方式 | 说明 |
|--------|------|------|
| 1 | 注解配置 | `@Component`、`@Service` 等 |
| 2 | Java 配置类 | `@Configuration` + `@Bean` |
| 3 | XML 配置 | 兼容使用，非首选 |

---

**总结：** Spring Boot 虽然强调"约定优于配置"、推崇自动化配置，但仍保留了对 XML 配置的完整支持。实际开发中建议以注解为主，在需要整合旧代码或特殊场景时可灵活引入 XML 配置。

---

## 3. Spring Boot 配置文件加载优先级你知道吗？

🟢 **简单** | 标签：Spring Boot, 后端

Spring Boot的配置文件加载遵循“高优先级覆盖低优先级”的原则，其加载顺序由高到低主要为：

1.  **命令行参数**：通过`--key=value`形式传入的参数优先级最高，便于临时覆盖配置。
2.  **外部配置文件**：位于应用外部（如jar包外）的`application-{profile}.yml`或`application.properties`，可按运行环境（如dev, prod）激活不同配置，且外部文件优先于内部。
3.  **内部配置文件**：位于`src/main/resources`下的`application-{profile}.yml`和默认的`application.yml`。
4.  **@PropertySource注解**：自定义加载的额外配置文件。
5.  **系统属性与环境变量**。

核心要点是：外部化配置优先于内部，profile专属配置优先于默认配置，显式指定的属性（如命令行）优先于文件。实践中，常通过外部配置文件和环境变量管理不同环境差异，保证应用灵活性。

---

## 4. Spring Boot 2.x 与 1.x 版本有哪些主要的改进和区别？

🟡 **中等** | 标签：Spring Boot, 后端

Spring Boot 2.x 相较于 1.x 版本是一次重大升级，主要区别体现在以下几个方面：

1.  **基础要求升级**：最低要求从 Java 7 提升至 Java 8，并全面拥抱 Java 9+ 的特性，如模块化支持。
2.  **核心框架更新**：基于 Spring Framework 5.x 构建，最重要的特性是支持了**响应式编程**（Reactive Stack），引入了 WebFlux 等组件，为开发非阻塞、高并发应用提供了新选择。
3.  **关键依赖与默认配置优化**：
    *   **数据库连接池**：将默认的 Tomcat JDBC 连接池更换为性能更优的 **HikariCP**。
    *   **依赖管理**：使用新的 `spring-boot-dependencies` 来管理依赖坐标，而非之前的 `spring-boot-dependencies` 资源文件，使依赖传递更清晰。
    *   **配置文件**：弃用了已作废的 `application.properties` 配置，全面采用 **`application.yml`**，并支持更多新的配置属性。
4.  **安全性与Actuator增强**：
    *   集成了 **Spring Security** 的更多新特性。
    *   **Spring Boot Actuator** 得到大幅改进：端点默认暴露方式从“宽松”改为“严格”（需显式配置），并改用 **Micrometer** 进行应用监控指标采集，提供了更丰富的度量集成。
5.  **内置容器与Web支持**：除了 Tomcat，更好地集成了 **Undertow** 和 **Jetty**。同时，Spring MVC 默认已适配 Servlet 4.0。
6.  **其他改进**：例如，嵌入式 Reactive Streams 支持、全新的错误处理自动配置、对 Kotlin 的更好支持等。

总结来说，Spring Boot 2.x 的核心变化是构建在 Spring 5 和 Java 8 之上，通过

---

## 5. Spring Boot 中 application.properties 和 application.yml 的区别是什么？ 

🟡 **中等** | 标签：Spring Boot, 后端

**文件格式与语法**
`application.properties` 采用传统的键值对格式（`key=value`），语法简单但表达层级结构时需用点号连接（如 `spring.datasource.url`）。`application.yml` 采用YAML格式，通过缩进表示层级关系，语法更简洁，支持列表和多行文本等复杂结构。

**可读性与维护性**
对于简单配置，两者差异不大。但在配置项众多、层级较深时，`.yml` 的缩进格式层次更分明，可读性通常更好。`.properties` 文件结构相对扁平，修改时容易因点号链条过长而出错。

**优先级与加载**
当两者同时存在于项目中（如在类路径下或同一目录），Spring Boot 会合并它们的配置，但 **同目录下的 `.properties` 文件优先级高于 `.yml` 文件**。若键冲突，`.properties` 中的值会覆盖 `.yml` 中的值。此外，`application-{profile}.properties/yml` 的profile专用配置文件会覆盖默认文件中的同名配置。

**核心共同点**
它们都是Spring Boot的外部化配置文件，用于替代`@Value`硬编码，支持属性占位符、多环境配置（通过`spring.profiles.active`）等核心功能，最终都会被加载到Spring的`Environment`中。

**如何选择**
选择哪一种主要取决于团队习惯和项目需求。对于新项目或配置结构复杂的情况，`.yml` 是更现代的选择；对于维护旧项目或追求与`.properties`一致的简单性，则可沿用`.properties`。两者可以混用，但建议在项目中统一风格以避免混淆。

---

## 6. Spring Boot 中如何实现异步处理？

🟡 **中等** | 标签：Spring Boot, 后端, Spring

## Spring Boot 中如何实现异步处理？

### 一、开启异步支持

在启动类或配置类上添加 `@EnableAsync` 注解：

```java
@SpringBootApplication
@EnableAsync
public class Application { }
```

### 二、使用 @Async 注解

在需要异步执行的方法上添加 `@Async`：

```java
@Service
public class AsyncService {
    @Async
    public void asyncTask() {
        // 异步执行的逻辑
    }
}
```

### 三、异步方法返回值

- **void**：无返回值，适合发送通知等场景
- **Future<T>**：可获取执行结果，支持取消操作
- **CompletableFuture<T>**：支持链式调用，功能更强大

```java
@Async
public CompletableFuture<String> asyncWithResult() {
    return CompletableFuture.completedFuture("结果");
}
```

### 四、自定义线程池

推荐自定义线程池替代默认的 SimpleAsyncTaskExecutor：

```java
@Configuration
public class AsyncConfig {
    @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

使用时通过 `@Async("taskExecutor")` 指定线程池。

### 五、注意事项

1. **避免自调用**：同类方法内部调用不会生效，因为绕过了代理
2. **异常处理**：实现 `AsyncUncaughtExceptionHandler` 处理无返回值方法的异常
3. **非 public 方法**：`@Async` 注解的方法必须是 public 的

---

## 7. Spring Boot 如何处理跨域请求（CORS）？

🟡 **中等** | 标签：Spring Boot, 后端

Spring Boot 处理跨域请求（CORS）主要通过以下方式实现：

**1. 注解方式（@CrossOrigin）**
可在控制器类或方法上添加 `@CrossOrigin` 注解，快速启用CORS支持。例如：
```java
@CrossOrigin(origins = "http://example.com", maxAge = 3600)
@RestController
public class ApiController { ... }
```
此方式灵活，可针对特定接口配置允许的源、方法、请求头等。

**2. 全局配置（WebMvcConfigurer）**
通过实现 `WebMvcConfigurer` 接口并重写 `addCorsMappings` 方法，可定义全局CORS策略：
```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "POST")
                .allowedHeaders("*")
                .maxAge(3600);
    }
}
```
此方法统一管理所有接口的跨域规则，适合全局控制。

**3. 过滤器方式（CorsFilter）**
对于需要更精细控制或结合安全框架（如Spring Security）时，可通过注册 `CorsFilter` Bean 实现：
```java
@Bean
public CorsFilter corsFilter() {
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    CorsConfiguration config = new CorsConfiguration();
    config.addAllowedOrigin("*");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");
    source.registerCorsConfiguration("/**", config);
    return new CorsFilter(source);
}
```
**核心要点**：CORS通过HTTP响应头（如`Access-Control-Allow-Origin`）实现。Spring Boot将其封装为易于配置的注解、全局设置或过滤器。实际开发中需根据安全要求合理设置允许的源、方法和头信息，避免使用通配符`*`暴露接口风险。若项目集成Spring Security，需确保CORS配置在安全过滤器链之前生效。

---

## 8. Spring Boot 打成的 jar 和普通的 jar 有什么区别 ?

🟡 **中等** | 标签：Spring Boot, 后端

Spring Boot 的 fat jar 与普通 jar 的主要区别如下：

**一、打包内容不同**
- **普通 jar**：只包含应用自身的 class 文件和资源文件，依赖的第三方 jar 需要在外部提供。
- **Spring Boot jar**：是一个可执行的 fat jar，包含应用代码、所有依赖 jar、内嵌的 Web 容器（如 Tomcat），是一个自包含的部署单元。

**二、内部结构不同**
Spring Boot jar 采用特殊目录结构：
- `BOOT-INF/classes/`：存放应用代码
- `BOOT-INF/lib/`：存放所有依赖 jar
- `org/springframework/boot/loader/`：存放启动器代码

**三、启动方式不同**
- **普通 jar**：需要手动构建 classpath，通过 `java -cp xxx.jar:lib/*` 启动
- **Spring Boot jar**：直接执行 `java -jar app.jar` 即可运行

**四、Manifest 配置不同**
- **普通 jar**：Main-Class 直接指向应用主类
- **Spring Boot jar**：Main-Class 指向 `JarLauncher`，由它负责设置 classpath，再调用应用的启动类

**五、核心优势**
Spring Boot fat jar 实现了"构建一次，到处运行"的理念，一个文件即可独立部署，简化了运维复杂度，非常适合容器化部署场景。

---

## 9. Spring Boot 支持哪些嵌入 Web 容器？

🟡 **中等** | 标签：Spring Boot, 后端

Spring Boot 默认支持并内嵌了三种主流的 Web 容器，它们分别是：

1.  **Apache Tomcat**：这是 Spring Boot 的默认嵌入式容器。它成熟、稳定、功能全面，拥有庞大的社区和丰富的文档支持。对于大多数 Web 应用，尤其是企业级应用，Tomcat 是一个可靠的选择。

2.  **Eclipse Jetty**：Jetty 以其轻量级和高效性著称，启动速度快，内存占用相对较低。它特别适合于需要微服务、云原生或嵌入到其他应用中（如中间件）的场景，对资源敏感型环境非常友好。

3.  **Undertow**：Undertow 是由 Red Hat 开发的高性能、非阻塞式 Web 服务器。它在高并发、低延迟场景下表现卓越，并且支持阻塞和非阻塞 API，提供了极大的灵活性。

Spring Boot 通过自动配置机制简化了容器的集成。开发者通常只需在 `pom.xml` 中排除默认的 `spring-boot-starter-web` 中的 Tomcat 依赖，并引入所需的 Jetty 或 Undertow starter 依赖，即可无缝切换，无需修改应用代码。这种“约定优于配置”的设计，充分体现了 Spring Boot 的灵活性和可扩展性，让开发者能够根据项目的性能需求和运维环境轻松选择最合适的 Web 容器。

---

## 10. SpringBoot 中如何实现定时任务 ?

🟡 **中等** | 标签：Spring Boot, 后端

SpringBoot通过`@Scheduled`注解与`@EnableScheduling`注解组合实现定时任务，步骤如下：

首先在启动类或配置类上添加`@EnableScheduling`注解，开启定时任务支持。接着在需要执行的方法上使用`@Scheduled`注解，并通过其属性指定执行策略，主要有三种方式：
1.  **固定间隔**：使用`fixedRate`（上一次开始执行后间隔多久）或`fixedDelay`（上一次执行完成后间隔多久），单位毫秒。
2.  **Cron表达式**：通过`cron`属性定义复杂的执行计划，如`@Scheduled(cron = "0 0/5 * * * ?")`表示每5分钟执行一次。
3.  **初始延迟**：配合`initialDelay`属性设置首次执行的延迟时间。

若需并行执行多个任务，避免任务阻塞，可在启动类额外添加`@EnableAsync`注解，并在任务方法上添加`@Async`注解使其异步执行。注意异步方法的返回值应为`void`或`Future`。定时方法还可获取当前执行时间参数`ScheduledContext`，用于日志记录或任务状态管理。

---

## 11. SpringBoot 默认同时可以处理的最大请求数是多少？

🟡 **中等** | 标签：Spring Boot, 后端

## SpringBoot 默认最大请求数

SpringBoot 默认内嵌 Tomcat 服务器，其并发处理能力主要由以下配置决定：

**核心参数**

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `server.tomcat.threads.max` | 200 | 最大工作线程数 |
| `server.tomcat.threads.min-spare` | 10 | 最小空闲线程数 |
| `server.tomcat.accept-count` | 100 | 等待队列长度 |

**处理机制**

当请求到达时，Tomcat 的处理流程如下：
1. 先由空闲线程直接处理
2. 线程不足时，新线程创建直到达到 max 上限（200）
3. 线程池满后，请求进入等待队列（最多100个）
4. 队列也满时，连接被拒绝

因此，理论最大并发请求数 = **200（处理中）+ 100（等待队列）= 300**

**配置调整**

```yaml
server:
  tomcat:
    threads:
      max: 500
      min-spare: 50
    accept-count: 200
```

**生产环境建议**

- 线程数并非越大越好，过多线程会导致上下文切换开销增加
- 应根据业务类型（CPU密集/IO密集）合理配置
- 高并发场景可考虑改用 Undertow 或 WebFlux（响应式编程）

面试回答时强调：**默认200个处理线程 + 100个等待队列**，并说明可以配置调整，体现对底层原理的了解。

---

## 12. SpringBoot（Spring）中为什么不推荐使用 @Autowired ？

🟡 **中等** | 标签：Spring Boot, 后端, Spring

在 Spring/SpringBoot 中，官方与社区更推荐使用**构造器注入**而非 `@Autowired` 进行字段注入，主要原因如下：

1.  **依赖明确，保证不可变性**：构造器注入强制在对象创建时就声明所有必需依赖，使依赖关系清晰、完整。通过将字段声明为 `final`，可保证核心依赖在对象生命周期内不可变，增强了代码的健壮性。

2.  **便于单元测试**：构造器注入的对象，其依赖通过构造函数传入。在测试时，可以方便地使用 `new` 操作符直接构造对象并传入模拟（Mock）的依赖，无需依赖 Spring 容器或反射机制，测试更简单、解耦。

3.  **避免循环依赖与空指针**：`@Autowired` 字段注入的对象在构造完成之后才被注入。若出现循环依赖，可能导致一方对象未完全初始化即被使用，引发 `NullPointerException`。构造器注入则能在启动阶段就暴露此类问题，促使开发者重构设计。

4.  **与设计原则契合**：构造器注入符合“接口设计”思想，使类成为对客户端代码更友好的、自描述的组件。

**总结**：虽然 `@Autowired` 使用便捷，但构造器注入在代码的可维护性、可测试性和设计严谨性上具有显著优势。因此，在编写新业务代码时，应优先考虑构造器注入。`@Autowired` 更适用于一些非核心的、可选的或遗留代码的集成场景。

---

## 13. 什么是 Spring Actuator？它有什么优势？

🟡 **中等** | 标签：Spring Boot, 后端

Spring Actuator 是 Spring Boot 提供的一个生产级监控与管理模块，它通过自动暴露一系列 HTTP 端点（如 `/health`, `/info`, `/metrics`），允许开发者实时洞察应用的运行状态、性能指标和内部细节，而无需手动编写大量监控代码。

其核心功能包括：健康检查（Health）、度量指标收集（Metrics）、环境信息查看（Info）、日志级别动态调整、审计事件跟踪以及与外部监控系统（如Prometheus、Grafana）的集成。开发者只需引入依赖并简单配置，即可获得开箱即用的监控能力。

它的主要优势在于：**1. 开箱即用**，极大降低了应用监控的接入成本；**2. 高度可定制与可扩展**，支持自定义健康指示器和指标，以满足特定业务监控需求；**3. 与Spring生态无缝集成**，是云原生和微服务架构中实现应用可观测性的标准实践，能有效支撑运维与故障排查。

---

## 14. 在 Spring Boot 中你是怎么使用拦截器的？

🟡 **中等** | 标签：Spring Boot, 后端

在Spring Boot中使用拦截器主要分为三个步骤：

**1. 创建拦截器**：创建一个类实现 `HandlerInterceptor` 接口。该接口定义了三个核心方法：
- `preHandle`：在Controller方法执行前调用，可进行权限校验、日志记录等，返回 `true` 继续执行，`false` 则中断。
- `postHandle`：在Controller方法执行后、视图渲染前调用。
- `afterCompletion`：在整个请求完成后调用，用于资源清理。

**2. 注册拦截器**：创建一个配置类，实现 `WebMvcConfigurer` 接口，并重写 `addInterceptors` 方法。通过 `InterceptorRegistry` 注册自定义的拦截器实例。

**3. 配置拦截路径与顺序**：
- 使用 `addPathPatterns()` 指定需要拦截的URL模式（如 `/api/**`）。
- 使用 `excludePathPatterns()` 排除不需要拦截的路径（如登录接口 `/login`）。
- 通过注册顺序控制多个拦截器的执行顺序。

**典型应用场景**：用于统一日志、登录鉴权（如检查Token）、性能监控（记录接口耗时）等横切关注点，有效实现代码解耦与复用。

---

## 15. 在 Spring Boot 中如何实现多数据源配置？

🟡 **中等** | 标签：Spring Boot, 后端

在Spring Boot中实现多数据源配置，核心是通过多个配置类分别创建并管理独立的数据源、事务管理器及对应的Mapper扫描。以下是关键步骤：

1. **配置文件**：在`application.yml`中分别定义多个数据源的连接信息（如`spring.datasource.master`和`spring.datasource.slave`）。

2. **配置类**：
   - 创建多个配置类，每个类使用`@Configuration`和`@MapperScan`注解，并指定不同的包路径用于扫描对应的Mapper接口。
   - 在每个配置类中，使用`@Bean`创建独立的`DataSource`、`SqlSessionFactory`和`TransactionManager`。
   - 使用`@Primary`注解指定主数据源，其他数据源用`@Qualifier`区分。

3. **数据源切换**（可选）：通过`AbstractRoutingDataSource`配合自定义注解和AOP实现动态数据源路由，适用于读写分离等场景。

示例配置类片段：
```java
@Configuration
@MapperScan(basePackages = "com.example.mapper.master", sqlSessionFactoryRef = "masterSqlSessionFactory")
public class MasterDataSourceConfig {
    @Bean(name = "masterDataSource")
    @Primary
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource masterDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "masterSqlSessionFactory")
    @Primary
    public SqlSessionFactory masterSqlSessionFactory(@Qualifier("masterDataSource") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        // 可在此配置MyBatis的mapper路径等
        return sessionFactory.getObject();
    }
}
```

通过上述方式，Spring Boot可以隔离管理多个数据源，确保各自事务和连接独立。

---

## 16. 如何在 Spring Boot 中定义和读取自定义配置？

🟡 **中等** | 标签：Spring Boot, 后端

在 Spring Boot 中定义和读取自定义配置主要有以下两种核心方式：

**1. 使用 `@Value` 直接注入属性值**
首先，在 `application.yml` 或 `application.properties` 中定义自定义配置项，例如：
```yaml
app:
  name: MyService
  version: 1.0
```
然后，通过 `@Value("${属性键}")` 注解将值直接注入到 Bean 的字段中：
```java
@Value("${app.name}")
private String appName;
```
此方式简单直接，适用于少量独立的配置项。

**2. 使用 `@ConfigurationProperties` 绑定对象（推荐）**
定义一个配置类，并使用 `@ConfigurationProperties` 注解指定属性前缀，Spring Boot 会自动将配置文件中的属性绑定到该类的字段上：
```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private String version;
    // getters and setters...
}
```
注入该 Bean 后即可通过对象方法访问。此方式结构清晰、类型安全，且支持属性校验，更适用于结构化的配置。

**补充**：对于多环境配置，可使用 `application-{profile}.yml` 文件并通过 `spring.profiles.active` 激活。若配置文件不在默认位置，可通过 `@PropertySource` 注解引入外部文件。

综上，简单配置用 `@Value`，复杂或需复用的配置推荐使用 `@ConfigurationProperties`，这是实现配置与业务代码解耦的最佳实践。

---

## 17. 如何在 SpringBoot 启动时执行特定代码？有哪些方式？

🟡 **中等** | 标签：Spring Boot, 后端, Spring

在 Spring Boot 中，执行启动时代码主要有以下几种方式：

1.  **实现 `CommandLineRunner` 接口**
    这是经典方式。实现该接口并重写 `run(String... args)` 方法，该方法将在 `ApplicationContext` 加载完成后、应用完全启动前执行。其参数为启动命令行参数。可以通过实现 `Ordered` 接口或使用 `@Order` 注解来控制多个 Runner 的执行顺序。

2.  **实现 `ApplicationRunner` 接口**
    与 `CommandLineRunner` 功能类似，但其 `run(ApplicationArguments args)` 方法的参数更强大，它将原始参数解析为 `ApplicationArguments` 对象，方便你通过 `getOptionNames()` 和 `getOptionValues()` 等方法访问具名参数（如 `--key=value`）和非选项参数。

3.  **使用 `@PostConstruct` 注解**
    将注解加在某个 Bean（通常是 `@Component`）的初始化方法上。该方法会在该 Bean 实例化、依赖注入完成后被调用。注意，它执行时机早于上述 Runner，且只影响单个 Bean 的初始化阶段。

4.  **监听特定事件（如 `ApplicationReadyEvent`）**
    通过创建监听器，监听如 `ApplicationReadyEvent`（应用完全就绪后）或 `ApplicationStartedEvent`（应用启动后）等事件。这种方式最为灵活，可精确控制代码执行的节点（如上下文刷新后、Web 服务器启动后等），并能访问完整的应用上下文。

**总结**：通用启动逻辑优先使用 `CommandLineRunner` 或 `ApplicationRunner`（后者参数处理更佳）。需要与 Bean 初始化紧密耦合的逻辑用 `@PostConstruct`。若需在特定生命周期节点（如应用就绪后）执行，则采用事件监听。

---

## 18. 说说你对 Spring Boot 事件机制的了解？

🟡 **中等** | 标签：Spring Boot, 后端

Spring Boot的事件机制基于Spring框架的ApplicationEvent和ApplicationListener，实现了组件间的松耦合通信。

其核心包含三部分：**事件**（继承ApplicationEvent）、**监听器**（实现ApplicationListener接口或使用@EventListener注解）和**事件发布器**（通过注入ApplicationEventPublisher发布事件）。

Spring Boot提供了一系列内置事件，如ApplicationStartingEvent、ApplicationReadyEvent等，方便在应用生命周期的各个阶段进行扩展。开发者也可以自定义事件，用于特定业务场景。

典型应用如：用户注册成功后，发布一个UserRegisteredEvent事件，由独立的监听器处理发送欢迎邮件、记录日志等后续操作，从而解耦核心业务逻辑。

此外，事件机制支持**异步处理**（通过@Async注解）和**事件监听顺序控制**（使用@Order注解），进一步提升了灵活性和性能。它是实现观察者模式和增强系统扩展性的重要手段。

---

## 19. Spring Boot 3.x 与 2.x 版本有哪些主要的改进和区别？ 

🔴 **困难** | 标签：Java, Spring Boot, 后端

Spring Boot 3.x 相较于 2.x 版本是具有里程碑意义的升级，主要区别和改进如下：

1.  **Java 版本基线提升**：要求 **Java 17** 或更高版本，不再支持 Java 8/11，推动开发者使用现代语言特性。
2.  **迁移至 Jakarta EE 9+**：底层依赖从 `javax.*` 迁移到 **`jakarta.*`** 命名空间。这是最显著的变更，所有 Servlet、Bean Validation 等 API 包名都需要更新。
3.  **全面拥抱 Spring Framework 6**：基于全新设计的 Spring 6，支持 AOT（预编译）与 GraalVM Native Image，可生成**原生可执行文件**，极大提升启动速度和降低内存占用。
4.  **可观测性（Observability）增强**：内置并优化了 **Micrometer Tracing** 集成，可无缝对接 OpenTelemetry 和 Zipkin，提供标准化的分布式追踪支持。
5.  **新的核心特性**：引入了如 **ProblemDetail** (RFC 7807) 作为统一的错误响应格式、声明式 **HTTP 接口客户端** 等新功能，提升了开发效率。

总结来说，3.x 是一次**底层架构的现代化升级**，带来了性能飞跃、云原生友好性以及开发体验的改进，但迁移成本也相对较高，主要涉及包名修改和适配新的编程范式。

---

