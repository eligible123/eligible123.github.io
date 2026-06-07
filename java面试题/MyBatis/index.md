---
title: MyBatis面试题
date: 2026-06-07
type: page
---

# MyBatis面试题

> 共 12 道面试题

## 1. MyBatis 的优点？

🟢 **简单** | 标签：MyBatis, 后端

MyBatis 是一款优秀的持久层框架，它通过将 SQL 语句与 Java 代码解耦，提升了数据库操作的灵活性和可维护性。其核心优点主要体现在以下几个方面：

首先，**灵活性高**。MyBatis 允许开发者直接编写原生 SQL 或使用动态 SQL 标签，便于针对复杂业务场景优化查询，避免了 ORM 框架如 Hibernate 的过度抽象，性能更可控。

其次，**易于集成和上手**。它通过简单的 XML 或注解进行配置，学习成本低，能与 Spring 等主流框架无缝整合，适合快速开发。

第三，**映射机制强大**。支持将查询结果灵活映射到 Java 对象，包括一对一、一对多等复杂关系，减少手动编码。

第四，**内置缓存机制**。提供一级缓存（会话级）和二级缓存（全局级），有效提升数据库访问性能。

此外，MyBatis 还具备**良好的可维护性**，SQL 集中管理便于团队协作和调试。总的来说，MyBatis 以简洁、高效的特点，在后端开发中广泛应用，特别适合对 SQL 有精细控制需求的项目。

---

## 2. MyBatis 自带的连接池有了解过吗？

🟢 **简单** | 标签：后端, MyBatis

MyBatis 内置了连接池管理功能，主要用于优化数据库连接的性能，避免频繁创建和销毁连接带来的开销。其核心实现是 `PooledDataSource`，它在 `UnpooledDataSource` 的基础上增加了连接池缓存。

其工作原理是：池中维护了一定数量的数据库连接。当应用需要获取连接时，优先从空闲连接列表中取；如果没有空闲连接且当前活跃连接数未达到上限，则创建新连接；若已达到上限，则进入等待队列。当连接使用完毕后，不会立即关闭，而是归还到池中，置为空闲状态供复用。

主要配置参数包括 `poolMaximumActiveConnections`（最大活跃连接数，默认10）和 `poolMaximumIdleConnections`（最大空闲连接数，默认5），以及连接超时时间等。

优势在于：1. **提升性能**：复用连接，减少网络和IO开销。2. **资源管理**：有效控制并发连接数，防止数据库连接被耗尽。

在实际生产中，虽然MyBatis自带连接池功能足够，但更常与第三方高性能连接池（如HikariCP、Druid）集成使用，因其提供更完善的监控、健康检查和扩展功能。是否启用内置连接池，可通过数据源配置（`type=”POOLED”`）来指定。

---

## 3. Mybatis 如何实现一对一、一对多的关联查询 ？

🟢 **简单** | 标签：后端, MyBatis

MyBatis主要通过两种方式实现关联查询：嵌套结果映射和嵌套查询。

**1. 嵌套结果映射（推荐）**  
通过一次SQL联表查询获取所有数据，在ResultMap中配置关联关系：
- **一对一**：使用`<association>`标签，将关联对象的属性直接映射到主对象。
- **一对多**：使用`<collection>`标签，将关联集合映射到主对象的集合属性。  
*优点：只需一次SQL查询，性能较高。*

**2. 嵌套查询**  
分步查询：先查询主表数据，再根据主表字段查询关联表。
- 在ResultMap中通过`select`属性指定第二次查询的ID，实现延迟加载。  
*注意：可能产生N+1查询问题，需谨慎使用。*

**示例配置**：
```xml
<!-- 嵌套结果映射 -->
<resultMap id="userOrderMap" type="User">
    <id property="id" column="user_id"/>
    <result property="name" column="name"/>
    <!-- 一对多 -->
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="amount" column="amount"/>
    </collection>
</resultMap>
```

**总结**：  
- 简单关联优先用嵌套结果映射，性能更优；  
- 需要懒加载时可使用嵌套查询；  
- 通过`association`和`collection`标签清晰映射对象关系。

---

## 4. 使用 MyBatis 的 mapper 接口调用时有哪些要求？

🟢 **简单** | 标签：Mybatis, 后端

使用 MyBatis 的 Mapper 接口调用时，主要有以下要求：

1. **接口与 XML 映射文件绑定**：Mapper 接口的全限定名必须与对应的 XML 映射文件中的 `namespace` 属性值完全一致。这是 MyBatis 将接口方法与 SQL 语句关联起来的基础。

2. **方法名对应 SQL ID**：接口中定义的方法名，必须与 XML 文件中对应 SQL 语句的 `id` 属性值相同。

3. **参数与结果映射**：
   - **参数传递**：方法的参数类型应与 XML 中 SQL 语句的 `parameterType` 匹配。多参数时，可使用 `@Param` 注解指定参数名，或按顺序引用（如 `#{0}`, `#{1}`）。
   - **返回类型**：方法的返回类型需与 SQL 语句的 `resultType` 或 `resultMap` 对应。返回单个对象、集合或基本类型，MyBatis 会自动处理映射。

4. **接口规范**：Mapper 接口应为 Java 接口，不能有实现类。MyBatis 在运行时通过动态代理机制为接口生成代理对象，因此调用时直接通过 `SqlSession.getMapper(YourMapper.class)` 获取实例即可，无需手动编写实现。

5. **注意事项**：接口中不能存在方法重载（即方法名相同但参数不同），因为 XML 中的 `id` 是唯一的。

总之，核心是保证 **“接口全限定名=XML命名空间”** 和 **“方法名=SQL语句ID”** 的一致性，并正确映射参数与返回结果，即可通过接口方式安全、简洁地调用数据库操作。

---

## 5. JDBC 编程有哪些不足之处，MyBatis 是如何解决的？

🟡 **中等** | 标签：后端, MyBatis

JDBC编程主要有以下不足：
1. **代码冗余**：每次操作数据库都需重复编写加载驱动、创建连接、创建语句对象、处理结果集、释放资源等步骤。
2. **硬编码与耦合**：SQL语句直接写在Java代码中，修改SQL需重新编译，不利于维护和优化。
3. **参数设置繁琐**：需手动设置SQL参数的索引和类型，容易出错且代码可读性差。
4. **结果集处理麻烦**：需手动遍历ResultSet，通过列名或索引获取数据并封装到对象中，重复且易出错。
5. **资源管理复杂**：需手动关闭Connection、Statement、ResultSet等资源，易引发资源泄漏。
6. **数据库移植性差**：不同数据库的SQL语法差异需在代码中处理，可移植性低。

MyBatis针对这些不足提供了解决方案：
- **SQL与代码分离**：将SQL写在XML配置文件或注解中，与Java代码解耦，便于统一管理和优化。
- **自动化映射**：通过配置或注解自动将结果集映射为Java对象，减少手动封装代码。
- **动态SQL**：使用OGNL表达式支持条件拼接、循环等动态SQL生成，适应复杂查询场景。
- **参数映射**：支持通过参数名或Map直接设置参数，无需手动管理索引和类型转换。
- **连接池与缓存**：内置连接池管理数据库连接，支持一级（Session级）和二级（应用级）缓存，提升性能。
- **简化资源管理**：框架自动管理资源生命周期，减少泄漏风险。

总之，MyBatis通过“配置/注解+动态SQL+自动映射”的模式，大幅减少了样板代码，提高了开发效率和可维护性。

---

## 6. MyBatis 写个 Xml 映射文件，再写个 DAO 接口就能执行，这个原理是什么？

🟡 **中等** | 标签：后端, MyBatis

MyBatis 的核心原理是基于动态代理机制。整个过程可以概括为：

1.  **XML 解析与注册**：MyBatis 启动时会解析所有配置的 XML 映射文件。它会将 XML 中定义的每个 SQL 语句（如 `<select>`、`<insert>` 等）与一个唯一的标识（通常是 Mapper 接口的全限定名 + 方法名）关联起来，并封装为一个 `MappedStatement` 对象，最终注册到全局配置（`Configuration`）中。

2.  **接口与动态代理**：当你定义一个 DAO（Mapper）接口时，MyBatis 并不会为它创建一个传统的实现类。相反，当获取该接口的实例（例如通过 `SqlSession.getMapper()`）时，MyBatis 会使用 JDK 动态代理，为该接口生成一个代理对象（`MapperProxy`）。

3.  **方法执行流程**：
    *   当调用代理对象的接口方法时，会被代理拦截器 `MapperProxy` 所捕获。
    *   代理对象根据接口的全限定名和当前被调用的方法名，构造出对应的 `MappedStatement` 的唯一 ID。
    *   通过该 ID，从全局配置中找到对应的 `MappedStatement`，从而获得预编译的 SQL 语句及其参数映射关系。
    *   最后，代理对象将调用委托给 `SqlSession`，由 `SqlSession` 负责真正执行 SQL（包括参数设置、语句执行、结果集映射等）。

简而言之，XML 文件提供了 SQL 的“原材料”和执行规则，而 DAO 接口定义了调用契约。MyBatis 通过动态代理在两者之间架起桥梁，将接口方法的调用透明地转化为对 XML 中定义 SQL 的执行。

---

## 7. MyBatis 动态 sql 有什么用？执行原理？有哪些动态 sql？

🟡 **中等** | 标签：后端, MyBatis

MyBatis 动态SQL的主要作用是：**根据运行时的不同参数和条件，动态地拼接和生成最终的SQL语句**。它解决了在Java代码中手动拼接SQL字符串时代码臃肿、易出错且难以维护的问题，实现了SQL逻辑与代码的分离，使映射文件更清晰、灵活。

其执行原理可以概括为：MyBatis使用基于OGNL的表达式引擎，在解析XML映射文件时，会将`<if>`, `<choose>`, `<where>`等动态SQL标签解析成一系列的`SqlNode`对象（如`IfSqlNode`, `ChooseSqlNode`）。在运行时，这些节点通过**组合模式**构成一个树形结构。当执行SQL时，MyBatis会遍历这个节点树，并结合传入的参数计算OGNL表达式，根据结果（true/false）决定包含或忽略对应的SQL片段，最终拼装出完整的SQL语句交给JDBC执行。

常用的动态SQL标签包括：
1.  **`<if>`**：最基础的条件判断，根据test属性的表达式决定是否包含SQL片段。
2.  **`<choose>`, `<when>`, `<otherwise>`**：类似于Java的`switch-case`语句，实现多分支选择。
3.  **`<where>`**：智能地处理WHERE子句，自动添加`WHERE`关键字并去除首个多余的`AND`或`OR`。
4.  **`<set>`**：用于更新操作，智能地处理SET子句，自动

---

## 8. MyBatis 如何实现数据库类型和 Java 类型的转换的？

🟡 **中等** | 标签：后端, MyBatis

MyBatis 主要通过 **TypeHandler** 接口实现数据库（JDBC）类型与 Java 类型的转换。

**核心机制：**
1.  **核心接口**：`TypeHandler` 负责两个方向的转换：在设置参数时，将 Java 类型转换为 JDBC 类型；在获取结果时，将 JDBC 类型转换为 Java 类型。
2.  **配置方式**：
    *   **显式指定**：在映射文件（XML）或注解中，通过 `jdbcType` 和 `javaType` 属性为参数或结果字段明确指定转换处理器。
    *   **自动识别**：MyBatis 会根据字段的 Java 类型和数据库字段的元数据（JDBC 类型）来推断并选择合适的处理器。
3.  **类型处理器注册表**：`TypeHandlerRegistry` 负责管理所有内置和自定义的 TypeHandler。它维护了一个映射关系，可以根据 Java 类型和 JDBC 类型的组合快速查找对应的处理器。

**扩展能力**：
对于内置处理器无法处理的场景（如枚举、JSON、自定义对象等），开发者可以通过实现 `TypeHandler` 接口或继承 `BaseTypeHandler` 类来自定义转换逻辑，并在配置中注册使用。

总之，MyBatis 通过一套可扩展的 TypeHandler 体系，提供了灵活且类型安全的双向转换能力，是连接对象世界与关系数据库的关键桥梁。

---

## 9. MyBatis 是否支持延迟加载？如果支持，它的实现原理是什么？

🟡 **中等** | 标签：后端, MyBatis

MyBatis **支持延迟加载**，这是一种优化策略，主要用于关联对象（一对一、一对多）的查询，避免一次性加载大量不必要的数据。

其**实现原理**基于**代理模式和拦截机制**：
1.  **按需加载**：当查询主对象（如用户）时，MyBatis 不会立即执行关联对象（如订单列表）的SQL查询。
2.  **创建代理**：它会为关联对象属性生成一个**代理对象**（如通过CGLIB或Javassist），并将其设置给主对象。
3.  **触发查询**：只有在程序**真正访问**该代理对象的属性或方法时，拦截器才会介入，触发预先存储的关联SQL语句，执行查询并完成数据封装。

**配置方式**：需在`mybatis-config.xml`全局开启`lazyLoadingEnabled`，并可设置`aggressiveLazyLoading`为`false`以实现更精细的控制。在映射文件中，也可通过`fetchType="lazy"`为特定关联指定延迟加载。

**优点**：显著提升性能，尤其适用于主表数据常用而关联数据不常用的场景，有效减少数据库交互次数。但需注意，不当使用可能导致大量分散的查询（N+1问题），需根据业务场景权衡。

---

## 10. MyBatis 的缺点？

🟡 **中等** | 标签：后端, MyBatis

MyBatis 的主要缺点包括以下几点：

**1. SQL 编写与维护成本较高**
MyBatis 需要手动编写大量 SQL 语句，当业务复杂时，SQL 会变得冗长且难以维护。同时，SQL 与代码耦合度高，若数据库表结构变更，需同步修改多处 XML 文件，增加了维护成本。

**2. 对开发人员要求较高**
需要熟悉 SQL 及性能优化技巧，编写不当易导致 N+1 查询、慢查询等问题。相比全自动 ORM（如 Hibernate），学习曲线较陡。

**3. 数据库移植性较差**
由于高度依赖特定数据库的 SQL 方言（如分页语法），更换数据库时需修改大量 SQL 映射文件，可移植性不如完全封装 SQL 的框架。

**4. 缓存机制存在局限**
二级缓存基于 namespace 粒度，容易导致脏数据；在分布式环境下，缓存同步需额外处理，使用不便。

**5. 功能相对简单**
缺乏 Hibernate 的对象状态自动追踪、延迟加载等高级特性，复杂关联查询需手动处理。

总体而言，MyBatis 适合对 SQL 有精细控制需求的场景，但在快速开发、简单 CRUD 的项目中可能显得繁琐。选择时应权衡灵活性与开发效率。

---

## 11. Mybatis 都有哪些 Executor 执行器？它们之间的区别是什么？

🟡 **中等** | 标签：Mybatis, 后端

MyBatis 中的 Executor 是核心组件，负责 SQL 语句的执行、缓存维护以及事务管理。它主要通过三种内置实现来工作：

1.  **SimpleExecutor（简单执行器）**：这是默认的执行器。其特点是**每执行一次 update 或 query，就开启一个新的 Statement 对象**，用完后立即关闭。实现简单，性能开销相对较大，因为它频繁地创建和销毁 Statement。
2.  **ReuseExecutor（可重用执行器）**：它对 Statement 做了缓存处理。**执行相同的 SQL 时，会复用之前创建好的 Statement 对象**，避免了重复创建的开销，从而提升了性能。
3.  **BatchExecutor（批处理执行器）**：它专门用于**将多个更新语句（update/insert/delete）组织成一个批次**，然后一次性提交给数据库执行。这能显著减少与数据库的交互次数，大幅提升批量操作的效率。

**核心区别**在于对 `java.sql.Statement` 对象的管理策略和适用场景：
*   **生命周期管理**：SimpleExecutor 每次新建；ReuseExecutor 复用；BatchExecutor 收集后统一处理。
*   **批量支持**：BatchExecutor 独有批量执行能力。
*   **适用场景**：默认 SimpleExecutor 即可；对同一SQL重复执行可用 ReuseExecutor 优化；大量数据增删改时，应选用 BatchExecutor 以获得最佳性能。

在 MyBatis 配置文件中，可通过 `<setting name="defaultExecutorType" value="..."/>` 来全局设置默认执行器类型。在实际开发中，尤其是数据迁移或批量导入场景，合理选用执行器能带来可观的性能提升。

---

## 12. 能详细说说 MyBatis 的执行流程吗？

🟡 **中等** | 标签：后端, MyBatis

MyBatis的执行流程可以概括为以下几个核心步骤：

**1. 加载配置与初始化**
程序启动时，MyBatis会读取核心配置文件（mybatis-config.xml）和所有的映射文件（Mapper XML），将SQL语句、参数映射、结果映射等信息解析并封装到一个全局的`Configuration`对象中，同时为每个SQL标签生成一个`MappedStatement`对象。最终，基于配置创建出全局唯一的`SqlSessionFactory`工厂。

**2. 获取会话与执行SQL**
应用通过`SqlSessionFactory.openSession()`方法获取一个`SqlSession`对象。`SqlSession`是与数据库交互的核心API。当调用如`selectList`、`insert`等方法时，执行流程如下：
*   **定位MappedStatement**：根据传入的Mapper接口方法或SQL的唯一ID，从`Configuration`中找到对应的`MappedStatement`对象。
*   **创建执行器**：根据配置创建`Executor`执行器（如`SimpleExecutor`、`ReuseExecutor`），它负责SQL的真正执行和缓存管理。
*   **创建处理器并执行**：`Executor`创建`StatementHandler`（负责JDBC Statement操作）、`ParameterHandler`（负责参数设置）和`ResultSetHandler`（负责结果集映射）。最终，通过JDBC的`PreparedStatement`执行SQL，并将数据库结果集通过`ResultSetHandler`映射成Java对象返回。

**3. 会话关闭与事务提交**
操作完成后，需要关闭`SqlSession`以释放资源。如果是更新操作，还需手动提交事务（或在配置中设置自动提交）。

整个流程体现了MyBatis将配置、SQL执行与结果映射解耦的设计思想，开发者只需关注SQL本身。

---

