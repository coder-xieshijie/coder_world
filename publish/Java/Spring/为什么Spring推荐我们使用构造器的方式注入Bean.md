



在Spring和Spring Boot中，**依赖注入**（Dependency Injection, DI）是核心概念之一，用于实现对象的松耦合。依赖注入的几种常见方式包括：

1. **构造器注入**（Constructor Injection）
2. **Setter注入**（Setter Injection）
3. **字段注入**（Field Injection）

虽然Spring支持多种注入方式，但推荐使用**构造器注入**。以下是几个推荐使用构造器注入的原因：

### 1. 不可变性和线程安全

使用构造器注入，可以确保所有依赖在对象创建时就被注入。这意味着对象在创建后是不可变的（immutable），从而提高了线程安全性。

```
public class MyService {
    private final SomeDependency dependency;

    @Autowired
    public MyService(SomeDependency dependency) {
        this.dependency = dependency;
    }
}
```

### 2. 强制依赖注入

构造器注入将依赖作为参数传递给构造器，因此在创建对象时，必须提供这些依赖，这防止了忘记注入某个必需依赖的情况。

```
public class MyService {
    private final SomeDependency dependency;

    // 强制在创建MyService时注入SomeDependency
    public MyService(SomeDependency dependency) {
        this.dependency = dependency;
    }
}
```

### 3. 简化单元测试

构造器注入使得单元测试变得更加简单和直观。你可以直接实例化对象并传递模拟的依赖，而不需要依赖Spring容器。

```
public class MyServiceTest {
    @Test
    public void testService() {
        SomeDependency mockDependency = mock(SomeDependency.class);
        MyService myService = new MyService(mockDependency);

        // 测试MyService的逻辑
    }
}
```

### 4. 避免字段注入的问题

字段注入（Field Injection）使用`@Autowired`直接注解在字段上，存在一些问题：

- **不方便测试**：在单元测试中，无法轻松地用模拟对象替换依赖。
- **隐藏依赖**：字段注入隐藏了类的依赖，阅读代码时不容易一目了然地看到该类所需要的所有依赖。
- **不可变性问题**：使用字段注入对象是可变的，依赖可以通过反射等手段改变。

字段注入示例：

```
public class MyService {
    @Autowired
    private SomeDependency dependency;
}
```



### 5. 多依赖自动注入

当使用构造器注入时，如果构造器只有一个且所有参数都可以由容器解析，Spring会自动进行依赖注入，这使得代码更简洁。

可以用lombok的**@RequiredArgsConstructor**代替@Autowired

```
public class MyService {
    // 有一个构造器且所有参数都可以由容器解析
    public MyService(SomeDependency dependency, AnotherDependency anotherDependency) {
        this.dependency = dependency;
        this.anotherDependency = anotherDependency;
    }
}
```

### 结论

构造器注入在Spring和Spring Boot中被推荐的原因包括不可变性和线程安全、强制依赖注入、简化单元测试、避免字段注入的问题，以及符合SOLID原则。通过使用构造器注入，可以编写出更健壮、可维护、易测试和松耦合的代码。