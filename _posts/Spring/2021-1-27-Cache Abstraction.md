---
layout: post
title: Cache Abstraction
categories:
- Spring
  description: Cache Abstraction、Cache
  keywords: 'Spring'、'Cache'
---

Spring框架提供了缓存的支持。Spring的Cache Abstraction对各种缓存方案提供了一致的使用方法，同业务代码解耦，减少了对业务代码的影响。

使用Cache Abstraction你只需要关注两个方面：
- 缓存声明：声明需要缓存操作的方法和策略。
- 缓存配置：用来存储和读取数据的后台缓存。

# 声明基于注解的Caching

对于缓存声明，Spring提供了一系列的注解：
- `@Cacheable`：缓存生成。
- `@CacheEvict`：缓存删除。
- `@CachePut`：在不干扰方法执行的情况下更新缓存。
- `@Caching`：将用于一个方法上的多个缓存重新分组。
- `@CacheConfig`：在类级别共享一些与缓存相关的设置。

## `@Cacheable`注解

使用该注解标注的方法，在第一次调用该方法之后会将方法的返回结果存储在缓存中。在后续的方法调用中直接从缓存中读取数据，而不需要执行方法。在最简单的情况下，该注解只需要一个属性用来表示和该方法相关联的缓存名称。如下所示：
```
@Cacheable("books")
public Book findBook(ISBN isbn) {...}
```
在这段代码中，findBook方法和名为`books`的缓存相关联。每次方法被调用时，会检查缓存中该方法是否已经被使用同样的请求参数调用过。通常情况下，只需要声明一个缓存，但该注解也可以声明多个缓存。如下所示：
```
@Cacheable({"books", "isbns"})
public Book findBook(ISBN isbn) {...}
```

### 默认Key生成器

由于缓存使用key-value方式存储，所以每次调用方法时需要生成一个合适的key。Caching Abstraction使用的默认`KeyGenerator`的逻辑如下：
- 如果没有参数，返回`SimpleKey.EMPTY`。
- 如果只有一个参数，则返回改实例。
- 如果超过一个参数，返回一个包含所有参数的SimpleKey。

只要参数实现了有效的`hashCode()`和`equals()`方法，这种key生成的方式就能有效地运行。如果不是这种情况，你需要改变key生成策略。

你可以通过实现`org.springframework.cache.interceptor.KeyGenerator`接口提供不同的key生成器。

### 自定义Key生成器的声明

由于缓存是通用的，而目标方法可能有复杂的签名导致该方法签名不能映射在缓存结构的顶部。当目标方法有多个参数但并不是每个参数都需要用来生成缓存的key时，这种问题就会变得十分明显。如下所示：
```
@Cacheable("books")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```
乍一看，两个`boolean`类型的参数影响了book查询的方式，但实际上他们可能对缓存没有用。

在这种情况下，可以在`@Cacheable`注解中指定`key`属性来配置如何生成缓存key。你可以使用SpEl表达或者调用任意的方法。推荐使用这种方式来覆盖默认的key生成器。因为随着代码库的增长，方法签名会变的越来越复杂。默认的key生成策略并不适用于所有方法。

下面例子展示了使用SpEL表达式声明key的方式：
```
@Cacheable(cacheNames="books", key="#isbn")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)

@Cacheable(cacheNames="books", key="#isbn.rawNumber")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)

@Cacheable(cacheNames="books", key="T(someType).hash(#isbn)")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

如果生成key的算法十分具体或者需要通用，你可以定义一个自定义的`keyGenerator`。
```
@Cacheable(cacheNames="books", keyGenerator="myKeyGenerator")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

*`key`和`keyGenerator`是互斥的，不能同时配置，否则会导致异常。*

### 默认的缓存解析器

Caching Abstraction使用`CacheResolver`通过使用配置的`CacheManager`来获取在方法上定义的缓存。

你可以通过实现`org.springframework.cache.interceptor.CacheResolver`接口来自定义缓存解析器。

### 自定义缓存解析器

默认的缓存解析器适合没有复杂解析要求和只有一个`CacheManager`的应用程序。

对于配置有多个`CacheManager`的应用程序，你可以为每个方法指定`CacheManager`，如下所示：
```
@Cacheable(cacheNames="books", cacheManager="anotherCacheManager") 
public Book findBook(ISBN isbn) {...}
```

你可为每个方法配置`CacheResolver`的方式来替换默认缓存解析器，如下所示：
```
@Cacheable(cacheResolver="runtimeCacheResolver") 
public Book findBook(ISBN isbn) {...}
```

*就像`key`与`keyGenerator`一样，`cacheManager`和`cacheResolver`也是互斥的。*

### 缓存同步

在多线程环境中，某个方法可能被使用相同的入参同时调用（特别是在程序刚启动时）。默认情况下，Cache Abstraction没用使用同步锁，同一个值会被多次计算。

对于这种多线程的情况，你可以使用`sync`属性告诉后台的缓存提供者在计算过程中去锁定cache entry。这样就只会有一个线性进行值的计算，其他线程会等待该线程执行完成后再访问缓存。
```
@Cacheable(cacheNames="foos", sync=true) 
public Foo executeExpensiveOperation(String id) {...}
```

### 有条件的缓存

有时，一个方法可能不需要对每次调用都进行缓存（比如，需要某些指定的入参符合一些条件时）。此时，我们可以使用`condition`属性。该属性接收一个`SPEL`表达式来判断`true`或者`false`。如果表达式值为`true`就执行缓存操作，否则不执行。例如，只有当`name`的长度小于32时才进行缓存操作：
```
@Cacheable(cacheNames="book", condition="#name.length() < 32") 
public Book findBook(String name)
```

除了`condition`属性，你可以使用`unless`属性来排除缓存中添加的值。`unless`的表达式会在方法调用完成后判断。例如，不向缓存中添加hardback的书籍：
```
@Cacheable(cacheNames="book", condition="#name.length() < 32", unless="#result.hardback") 
public Book findBook(String name)
```

Cache Abstraction支持`java.util.Optional`，只有`isPresent`为`true`时才会进行缓存。`#result`总是指向业务实体，所以上一个例子可以重写成下面的例子：
```
@Cacheable(cacheNames="book", condition="#name.length() < 32", unless="#result?.hardback")
public Optional<Book> findBook(String name)
```
`result`仍然指向`Book`而不是`Optional`。由于它可能是`null`，我们应该使用安全的访问操作符。

### 可以使用的SpEL表达式Context

- `methodName`：被调用的方法名称。例：`#root.methodName`
- `method`：被调用的方法。例：`#root.method.name`
- `target`：被调用的目标对象。例：`#root.target`
- `targetClass`：被调用目标的class。例：`#root.targetClass`
- `args`：调用目标方法时的参数数组。例：`#root.args[0]`
- `caches`：当前正在执行的方法的缓存集合。例：`#root.caches[0].name`
- 参数名称：任意一个方法参数的名称。如果名称访问不到，可以使用`#a<#arg>`，其中`#arg`代表参数的索引（从0开始）。例：`#iban`或者`#a0`。
- `result`：方法调用的结果（需要缓存的值）。只能在`unless`、`cache put`（用来计算key）、`cache evict`（当`beforeInvocation`为`false`时）表达式中使用。对于支持的包装类（如`Optional`），`#result`指向实际的对象，而不是包装类。例：`#result`

## `@CachePut`注解

当缓存需要在不干扰方法执行的情况下更新时，你可以使用`@CachePut`注解。就是说，使用该注解的方法总是会被调用并且将方法的结果放入缓存。它支持和@Cacheable相同的属性选项，用于对缓存的填充。下面的例子展示了如何使用该注解：
```
@CachePut(cacheNames="book", key="#isbn")
public Book updateBook(ISBN isbn, BookDescriptor descriptor)
```

## `@CacheEvict`注解

和`@Cacheable`相反，`@CacheEvict`用来标识执行删除缓存的方法。`CacheEvict`和前面的注解一样，需要指定一个或多个缓存对象，允许自定义缓存和key的解析或者配置condition条件。同时提供了一个额外的参数`allEntries`用来删除所有缓存对象。下面的例子删除了`books`缓存中的所有对象：
```
@CacheEvict(cacheNames="books", allEntries=true) 
public void loadBooks(InputStream batch)
```

使用`beforeInvocation`属性用来指定在调用方法前还是调用方法后删除缓存。当方法执行的结果和删除的缓存没有关系时可以在调用方法前进行删除，否则，建议在方法调用后进行删除。（如果在执行前删除，在方法执行过程中调用了写入缓存的方法，那么对应的缓存可能不会被更新而和删除前的缓存一样。）

## `@Caching`注解

有时，一个方法上需要配置多个注解（例如`@CacheEvict`和`@CachePut`），因为不同的缓存对象的condition或者key表达式不一样。`@Caching`注解允许`@Cacheable`、`@CacheEvict`、`@CachePut`在同一个方法上使用。例如：
```
@Caching(evict = { @CacheEvict("primary"), @CacheEvict(cacheNames="secondary", key="#p0") })
public Book importBooks(String deposit, Date date)
```

## `@CacheConfig`注解

到目前为止，我们已经看到了前面的缓存操作提供了需要自定义选项。然而，如果我们再每个方法上都配置这些自定义选项会闲着冗长。例如，为一个类的每个缓存操作配置一个缓存名称。此时，我们可以使用`@CacheConfig`注解。线面的例子使用该注解配置缓存名称：
```
@CacheConfig("books") 
public class BookRepositoryImpl implements BookRepository {

    @Cacheable
    public Book findBook(ISBN isbn) {...}
}
```
`@CacheConfig`是一个类级别的注解，改注解可以配置缓存名称、自定义`KeyGenerator`，自定义`CacheManage`，自定义`CachaResolver`。但该注解不执行任何缓存操作。

方法级别的配置会覆盖该注解的配置。因此，每个缓存操作的自定义配置有三个级别：
- 全局配置，可以获取`KeyGenerator`，`CacheManage`。
- 类级别配置，使用`@CacheConfig`。
- 方法级别的配置。

## 开启缓存

在Spring的应用程序中，你需要在任意一个`@Configuration`修饰的类上添加注解`@EnableCaching`来开始缓存：
```
@Configuration
@EnableCaching
public class AppConfig {
}
```

## 自定义注解

你可以使用`@Cacheable`,`@CachePut`, `@CacheEvict`, `@CacheConfig`等注解作为元注解。例如，我们使用自己声明的注解替换@Cacheable`：
```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Cacheable(cacheNames="books", key="#isbn")
public @interface SlowService {
}
```

现在我们替换下面的代码：
```
@Cacheable(cacheNames="books", key="#isbn")
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

下面的例子为替换后的代码：
```
@SlowService
public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed)
```

# 在Spring Boot中配置缓存

在Spring Boot应用中只要你使用了`@EnableCaching`开启了缓存，Spring Boot就会自动为你配置缓存的组件。



## 缓存提供者

Cache Abstraction不提供具体的存储而是依赖于`org.springframework.cache.Cache`和`org.springframework.cache.CacheManager`接口的实现。

如果你没有定义一个类型为`CacheManager`或者名字为`cacheResolver`的`CacheResolver`时，Spring Boot尝试检测以下缓存提供者（按照指定的顺序）：
- Generic
- JCache (JSR-107) (EhCache 3, Hazelcast, Infinispan, and others)
- EhCache 2.x
- Hazelcast
- Infinispan
- Couchbase
- Redis
- Caffeine
- Simple

如果`CacheManager`是Spring Boot自动配置的，你可以通过暴露一个实现在了`CacheManagerCustomizer`接口的bean在它初始化之前进一步调整配置。下面的例子设置了一个标志允许`null`值传递到底层的映射中：
```
@Bean
public CacheManagerCustomizer<ConcurrentMapCacheManager> cacheManagerCustomizer() {
    return new CacheManagerCustomizer<ConcurrentMapCacheManager>() {

        @Override
        public void customize(ConcurrentMapCacheManager cacheManager) {
            cacheManager.setAllowNullValues(false);
        }

    };
}
```
上面的例子中，需要一个自动配置的`ConcurrentMapCacheManager`。如果你使用别的自动配置的缓存提供者，这个配置将不会生效。

### Redis配置

如果已经配置了Redis，那么`RedisCacheManager`会被自动配置。下面配置两个缓存名称，缓存存活时间为10分钟：
```
spring.cache.cache-names=cache1,cache2
spring.cache.redis.time-to-live=10m
```
