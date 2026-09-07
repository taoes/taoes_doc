Spring 核心框架大量使用设计模式，高频这几个：

1. **工厂模式**：BeanFactory、ApplicationContext，Bean 工厂，负责 Bean 实例创建，把对象创建和业务使用解耦。
2. **单例模式**：Spring 默认 Bean 作用域 singleton，容器内只创建一个实例（是容器级单例，不是 Java 原生单例）。
3. **代理模式**：AOP 核心，JDK 动态代理 / CGLIB 代理，对目标方法增强，不修改原有代码。
4. **装饰器模式**：比如`BeanWrapper`、各种 Resource 包装类，对原有对象层层增强，不改变原对象接口。
5. **适配器模式**：`HandlerAdapter`，适配不同 Controller（HttpRequestHandler、Controller 等），统一调度。
6. **策略模式**：Bean 实例化策略、资源加载策略；还有事务管理器不同实现，可切换。
7. **模板方法模式**：JdbcTemplate、RestTemplate，固定主流程，可变部分交给回调实现。
8. **观察者模式**：Spring 事件监听`ApplicationEvent`、`ApplicationListener`，发布订阅。

[[../../框架应用/Spring/设计模式|设计模式]]