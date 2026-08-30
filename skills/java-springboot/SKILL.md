---
name: java-springboot
description: 获取 best practices for developing applications with Spring Boot.
---

# Spring Boot Best Practices

Your goal is 迁移到 help me 写入 high-quality Spring Boot applications by following established best practices.

## 项目 设置 & Structure

- **构建 Tool:** Use Maven (`pom.xml`) or Gradl`build.gradle````) for 依赖 management.
- **Starters:** Use Spring Boot starters (e.g., `spring-boot-starter-web`, `spring-boot-st`spring-boot-st`rter-ddata-jpaepende`rter-data-jpa`
- **包 Structure:** Organize code by feature/domain (e.g., `com.example.app.order`, `com.示例.`com.`com.示例.`pp.`com.示例.`layer ` by layer (`e.g., `ontroll`com.示`ontroll`t`com.示例.`com.example.app.controller``).`

## 依赖 Injection & Components

- **Constructor Injection:** Always use constructor-based injection for required 依赖. This makes components easier 迁移到 测试 and 依赖 explicit.
- **Immutability:** Declare 依赖 fields as `private final`.
- **组件 Stereotypes:** Use `@Component`, `@__CO__COD__C`@`CODE_5__@`e``@`@Co```@Control`stController`estController`stController`ions appropriately 迁移到 define beans.

## 配置

- **Externalized 配置:** Use `application.yml` (or `applic`applic`tion`tion.properties`uration. YAML is often preferred for its readability and hierarchical structure.
- **类型-Safe Properties:** Use `@ConfigurationProperties` 迁移到 bind 配置 迁移到 strongly-typed Java objects.
- **Profiles:** Use Spring Profiles (`application-dev.yml`, `applicatio`applicatio`-prod.yml` e`-prod.yml`specific configurations.
- **Secrets Management:** Do not hardcode secrets. Use 环境 variables, or a dedicated secret management tool like HashiCorp Vault or AWS Secrets Manager.

## Web Layer (Controllers)

- **RESTful APIs:** 设计 清空 and consistent RESTful endpoints.
- **DTOs (Data Transfer Objects):** Use DTOs 迁移到 expose and consume data in the API layer. Do not expose JPA entities directly 迁移到 the 客户端.
- **Validation:** Use Java Bean Validation (JSR 380) with annotations (`@Valid`CODE_1__`@Size`CODE_2__`e`e`) on DTOs 迁移到 验证 请求 payloads.
- **错误 Handling:** 实现 a global exception 处理器 using `@ControllerAdvice` and `@Excepti`@Excepti`nHandler`nHandler`tent 错误 responses.

## 服务 Layer

- **Business Logic:** Encapsulate all business logic within `@Service` classes.
- **Statelessness:** Services 应该 be stateless.
- **交易 Management:** Use `@Transactional` on 服务 methods 迁移到 管理 数据库 transactions declaratively. Apply it at the most granular level necessary.

## Data Layer (Repositories)

- **Spring Data JPA:** Use Spring Data JPA repositories by extending `JpaRepository` or `Crud`Crud``epository`tandard 数据库 operations.
- **Custom Queries:** For complex queries, use `@Query` or the JPA Criteria API.
- **Projections:** Use DTO projections 迁移到 获取 only the necessary data from the 数据库.

## Logging

- **SLF4J:** Use the SLF4J API for logging.
- **Logger Declaration:** `private static final Logger logger = LoggerFactory.getLogger(MyClass.class);`
- **Parameterized Logging:** Use parameterized messages (`logger.info("Processing user {}...", userId);`) instead of 字符串 concatenation 迁移到 improve 性能.

## Testing

- **单位 Tests:** 写入 单位 tests for services and components using JUnit 5 and a mocking 框架 like Mockito.
- **Integration Tests:** Use `@SpringBootTest` for integration tests that load the Spring 应用 context.
- **测试 Slices:** Use 测试 slice annotations like `@WebMvcTest` (for controllers) or `@D__CODE`taJpaTest`st`or repositories) 迁移到 测试 specific parts of the 应用 in isolation.
- **Testcontainers:** Consider using Testcontainers for reliable integration tests with real databases, message brokers, etc.

## Security

- **Spring Security:** Use Spring Security for 认证 and 授权.
- **Password 编码:** Always encode passwords using a strong 哈希 algorithm like BCrypt.
- **输入 清理:** Prevent SQL injection by using Spring Data JPA or parameterized queries. Prevent Cross-景 Scripting (XSS) by properly 编码 输出.