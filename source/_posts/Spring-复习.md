---
title: Spring 复习
date: 2021-04-04 00:46:54
categories: Spring
tags: Spring
---

本文是对 Spring 的一些总结，不定时更新

<!-- more -->

# Spring

## 介绍

我们一般说 Spring 框架指的都是 Spring Framework，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发。这些模块是：核心容器、数据访问 / 集成，、Web、AOP（面向切面编程）、工具、消息和测试模块。比如：Core Container 中的 Core 组件是 Spring 所有组件的核心，Beans 组件和 Context 组件是实现 IOC 和依赖注入的基础，AOP 组件用来实现面向切面编程。

## IOC

IoC（Inverse of Control: 控制反转）是一种**设计思想**，就是 **将原本在程序中手动创建对象的控制权，交由 Spring 框架来管理。** IoC 在其他语言中也有应用，并非 Spirng 特有。 **IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value）,Map 中存放的是各种对象。**

将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件 / 注解即可，完全不用考虑对象是如何被创建出来的。** 在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

### FactoryBean 与 BeanFactory

**`BeanFactory` 是 Spring 的顶级容器接口，定义了 Bean 工厂的基本功能特性**，并且 Spring 要求所有 IOC 容器都要实现 `BeanFactory` 接口。`BeanFactory` 负责管理 `bean`。

**FactoryBean是个 bean**，在IOC容器的基础上给Bean的实现加上了一个简单工厂模式和装饰模式，**是一个可以生产对象和装饰对象的工厂bean**，由spring管理后，生产的对象是由 getObject() 方法决定的。

### bean 的作用域

- **singleton** : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- **prototype** : 每次请求都会创建一个新的 bean 实例。
- **request** : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- **session** : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。
- **global-session**： 全局 session 作用域，仅仅在基于 portlet 的 web 应用中才有意义，Spring5 已经没有了。Portlet 是能够生成语义代码 (例如：HTML) 片段的小型 Java Web 插件。它们基于 portlet 容器，可以像 servlet 一样处理 HTTP 请求。但是，与 servlet 不同，每个 portlet 都有不同的会话

### 注入 bean

- set 方法注入
- 构造器注入
- 静态工厂注入
- 实例工厂注入

```xml
<bean id="myCar" class="cn.tewuyiang.pojo.Car">
    <property name="speed" value="100"/>
    <property name="price" value="99999.9"/>
</bean>
<bean id="myCar" class="cn.tewuyiang.pojo.Car">
    <constructor-arg name="speed" value="100" />
    <constructor-arg name="price" value="99999.9"/>
</bean>
<bean id="car" class="cn.tewuyiang.factory.SimpleFactory" factory-method="getCar"/>
<bean id="factory" class="cn.tewuyiang.factory.SimpleFactory" />
<bean id="car" factory-bean="factory" factory-method="getCar" />
```

### 注入接口多个实现类

当一个注入接口有多个实现类时，会报错。因为 `@Autowired` 是按照类型注入的。当要注入的类型在容器中存在多个时，Spring 不知道要注入哪个实现类，所以会报错。

可以按照 name 注入，使用 `@Resource` 或者 `@Qualifier`

- `@Resource`：可以通过 byName 和 byType 注入，默认按 byName，匹配不到再按 byType
- `@Qulifier`：按名称注入，和 Autowired 一起使用

```java
@Service("dogImpl")
public class DaoImpl impliments IAnimal{}

public class AnimalController {
    @Resource(name="dogImpl")
    private IAnimal dogImpl;
	//@Qualifier("DaoImpl")
    //private IAnimal dogImpl;
}
```

### 注入集合

- 注入 List<A> 会将所有实现 A 接口的实现类加入集合
-  注入 Map<String, A> 会将所有实现 A 接口的实现类加入集合，key 是名字

### 初始化过程

**1. 资源定位解析加载与注册**

在 Java 中，资源被抽象为 URL，而 Spring 中将物理资源的访问方式抽象为 Resource。

![Spring IoC的初始化过程](https://images.xiaozhuanlan.com/photo/2019/57da0deca924d0e73dbb56501d2ec4be.png)

以 xml 配置的配置文件的解析工作是在 `refresh()` 方法的 `obtainFreshBeanFactory` 方法中进行的。该方法对 xml 进行解析，将资源文件包装为 Resource 资源，并将 bean 解析为 `BeanDefinition` 对象，注册到容器中，最后返回一个 beanFactory 工厂 `DefaultListableBeanFactory`。

### Bean 的生命周期

<img src="https://cdn.jsdelivr.net/gh/crwen/img/blog/bean_lifecycle.png" style="zoom:80%;" />

1. Bean 资源定位，加载读取配置并解析，最后将解析的 BeanDefinition 放在一个 Map 中
2. 遍历 Map，利用反射实例化 Bean
3. 属性赋值 populateBean()
4. 如果 Bean 实现了 Aware 接口，调用相应方法（setBeanName/factory/...）
5. 如果实现了 BeanPostProcessor，调用`BeanPostProcess#postProcessBeforeInitialization`
6. 初始化 init()-method()
7. 如果实现了 BeanPostProcessor，`BeanPostProcess#postProcessAfterInitialization`
8. 销毁 DiposibleBean#destory()
9. destor-method()

### 循环依赖

- **singletonFactories** ： 进入实例化阶段的单例对象工厂的 cache （三级缓存）
- **earlySingletonObjects** ：完成实例化但是尚未初始化的，提前暴光的单例对象的 Cache （二级缓存）
- **singletonObjects**：完成初始化的单例对象的 cache（一级缓存）

A 引用 B，B 引用 A

调用 getBean() 方法获取 A 对象时，先回从缓存中取。如果缓存中没有的话，就创建 A 的实例，然后会将 A 放入 `singletonFactories`  缓存，填充 A 的属性时发现 A 引用 B，于是会先调用 getBean() 方法创建 B。

创建 B 的实例后，将其放入 `singletonFactories`  缓存，然后填充 B 的属性时发现 B 又引用 A，又会调用 getBean() 方法获取 A。

这时，可以从`singletonFactories`  缓存中获取到 A，然后将其放入 `earlySingletonObjects` 缓存，返回给 B，继续 B 的属性填充过程。

B 创建完成后，会将其放入 `singletonObjects` 中。然后 A 会继续创建，这时。

## AOP

Aop 即面向切面编程，简单地说就是**将代码中重复的部分抽取出来，在需要执行的时候使用动态代理的技术，在不修改源码的基础上对方法进行增强。优点是可以减少代码的冗余，提高开发效率，维护方便**。Spring 会根据类是否实现了接口来判断动态代理的方式，如果实现了接口会使用 JDK 的动态代理，核心是 InvocationHandler 接口和 Proxy 类，如果没有实现接口会使用 cglib 的动态代理，cglib 是在运行时动态生成某个类的子类，如果某一个类被标记为 final，是不能使用 cglib 动态代理的。

```java
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config)) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        return new ObjenesisCglibAopProxy(config);
    }
    else {
        return new JdkDynamicAopProxy(config);
    }
}
```

**AspectJ 与 Spring AOP**

Spring AOP 属于运行时增强，而 AspectJ 时编译时增强。Spring AOP 基于代理，而 Aspect 基于字节码操作

### 术语

- Joinpoint (连接点): 指那些被拦截到的点，在 spring 中这些点指的是方法，因为 spring 只支持方法类型的连接点。例如业务层实现类中的方法都是连接点。
- Pointcut (切入点): 指我们要对哪些 Joinpoint 进行拦截的定义。例如业务层实现类中被增强的方法都是切入点，切入点一定是连接点，但连接点不一定是切入点。
- Advice (通知 / 增强): 指拦截到 Joinpoint 之后所要做的事情。
- Introduction (引介): 引介是一种特殊的通知，在不修改类代码的前提下可以在运行期为类动态地添加一些方法或 Field。⑤Weaving (织入): 是指把增强应用到目标对象来创建新的代理对象的过程。spring 采用动态代理织入，而 AspectJ 采用编译期织入和类装载期织入。
- Proxy（代理）: 一个类被 AOP 织入增强后，就产生一个结果代理类。
- Target (目标): 代理的目标对象。⑧Aspect (切面): 是切入点和通知（引介）的结合。

### 注解

- `@Before` 前置通知
- `@AfterThrowing` 异常通知
- `@AfterReturning` 后置通知
- `@After` 最终通知
- `@Around` 环绕通知。最终通知会在后置通知之前执行，为解决此问题一般使用环绕通知。

### 动态代理

有接口：JDK 动态代理，反射 Proxy、InvocationHandler

无接口：GClib，字节码技术创建代理对象

**JDK 动态代理步骤**

1. 编写需要被代理的类的接口
2. 编写代理类，需要实现 `InvocationHandler` 接口，重写 `invoke` 方法
3. 使用 `Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)` 动态创建代理对象，通过代理类对象调用业务方法

```java
interface DemoInterface {
    String hello(String msg);
}

class DemoImpl implements DemoInterface {
    @Override
    public String hello(String msg) {
        System.out.println("msg = " + msg);
        return "hello";
    }
}
```

```java
class DemoProxy implements InvocationHandler {

    private DemoInterface service;

    public DemoProxy(DemoInterface service) {
        this.service = service;
    }

    @Override
    public Object invoke(Object obj, Method method, Object[] args) throws Throwable {
        System.out.println("调用方法前...");
        Object returnValue = method.invoke(service, args);
        System.out.println("调用方法后...");
        return returnValue;
    }
}
```

```java
public class Solution {
    public static void main(String[] args) {
        DemoProxy proxy = new DemoProxy(new DemoImpl());
        DemoInterface service = Proxy.newInstance(
            DemoInterface.class.getClassLoader(),
            new Class<?>[]{DemoInterface.class},
            proxy
        );
        System.out.println(service.hello("呀哈喽！"));
    }
}
```

### 原理

Spring AOP 是由 BeanPostProcessor 后置处理器处理的。在创建实例和初心填充之后，会在 `initializeBean` 中调用 后置处理器的 `postProcessAfterInitialization` 方法。AOP 在这里对创建了代理对象，对Bean 进行了替换

JDKDynamicAopProxy 实现了 InvocationHandler 接口，在 invoke 方法中会获取拦截器链，如果有拦截器就会创建 MethodInvocation 并调用 proceed 方法，否则直接反射调用目标方法。因此 Spring AOP 对目标对象的增强是通过拦截器实现的。

## MVC

### 组件

- **前端控制器 DispatcherServlet**：是整个流程控制的核心，负责接收请求并转发给对应的处理组件。
- **处理器映射器 HandlerMapping**：完成URL 到 Controller映射的组件，DispatcherServlet 接收到请求之后，通过 HandlerMapping 将不同的请求映射到不同的 Handler
- **处理器适配器 HandlerAdapter**：DispatcherServlet 通过 HandlerAdapter 来执行不同的 Handler。
- **视图解析器 ViewResolver**：DispatcherServlet 通过它将逻辑视图解析为物理视图，最终将渲染的结果响应给客户端。

### 原理

![img](https://img-blog.csdnimg.cn/20200221001135320.png)

1. 请求发送到前端控制器 DispatcherServlet
2. 前端控制器接收到请求后会使用 Handler Mapping 进行处理。HandlerMapping 会查找到具体处理请求的 Handler 对象
3. HandlerMapping 找到对应的 Handler 之后，返回一个 Handler 执行链（HandlerExecutionChain），在这个执行链中包括了拦截器和处理请求的 Handler。HandlerMapping 将这个执行链返回给前端控制器
4. 前端控制器接收到执行链之后，会调用 Handler 适配器去执行 Handler
5. Handler 适配器执行完 Handler 之后会得到一个 ModelAndView，并返回给前端控制器
6. 前端控制器接收到 HandlerAdapter 返回的 ModelAndView 之后，会根据其中的视图名调用视图解析器
7. 视图解析器根据视图名解析成一个真正的 View 视图，并返回给前端控制器
8. 前端控制器接收到视图之后，会根据 ModelAndView 中的 model 对视图数据的进行填充/渲染
9. 渲染完成之后，前端控制器将结果返回给客户端

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    // 根据当前 request 获取 handler 执行链
    // handler 包含了请求 url，以及对应的 controller 以及 controller 中的方法
    mappedHandler = getHandler(processedRequest);
    // 通过 handler 获取对应的适配器，adapter 负责完成参数解析
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

    // 遍历所有定义的 interceptor，执行 preHandler 方法
    if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
    }

    // 实际调用 Handler 的地方
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    // 处理默认视图名，也就是添加前缀和后缀等
    applyDefaultViewName(processedRequest, mv);
    // 遍历所有定义的 interceptor，执行 preHandler 方法
    mappedHandler.applyPostHandle(processedRequest, response, mv);
}
// 处理最后的结果
processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
}
```

### 建立请求映射

Web 容器会在 onRefresh 中初始化一些配置

```java
protected void initStrategies(ApplicationContext context) {
    initHandlerMappings(context); // 初始化 HandlerMapping
    initHandlerAdapters(context); // 初始化 HandlerAdapter
    initHandlerExceptionResolvers(context); // 初始化异常处理器
    initRequestToViewNameTranslator(context);
    initViewResolvers(context); // 初始化视图解析器
    initFlashMapManager(context);
}
```

初始化 HandlerMappings 时会将 RequestMappingHandlerMapping 加入集合中，这个类实现了 `InitializingBean`，在初始化时会调用 `afterPropertiesSet` 方法，初始化请求方法的映射。初始化时会获取 Controller 修饰的类，然后扫描类中的方法，将被 RequestMapping 修饰的方法的信息封装成一个 mapping，然后放入一个 map 中。

```java
public void register(T mapping, Object handler, Method method) {
    HandlerMethod handlerMethod = createHandlerMethod(handler, method);
    this.mappingLookup.put(mapping, handlerMethod); // 注册 RequestMappingInfo 和 HandlerMethod
    // 注册请求路径与对应的 RequestMappingInfo
    List<String> directUrls = getDirectUrls(mapping);
    for (String url : directUrls) {
        this.urlLookup.add(url, mapping); 、// <url, list>
    }

    if (getNamingStrategy() != null) {
        name = getNamingStrategy().getName(handlerMethod, mapping);
        addMappingName(name, handlerMethod);
    }
    // 创建及注册 MappingRegistation 信息
    this.registry.put(mapping, new MappingRegistration<>(mapping, handlerMethod, directUrls, name));

}
```

当请求到来时，会调用 RequestMappingHandlerMapping 的 getHandler() 方法，从map 中根据请求路径找到对应的 Handler

```java
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
    // 根据 request 获取 handler
    Object handler = getHandlerInternal(request);

    // 封装 Handler 执行链
    HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
    return executionChain;
}
```

```java
public List<T> getMappingsByUrl(String urlPath) {
    return this.urlLookup.get(urlPath);
}
```

### 拦截器

请求到来时，前端控制器 DispatcherServlet 会将使用 HandlerMapping 来找到执行请求的 Handler。在得到 Handler 的过程中，会将我们配置的拦截器加入到执行链中。获取到 HanlderAdapter 后，会遍历所有遍历的 interceptor，执行 preHandle 方法。组装完视图后，会调用拦截器的 postHandler() 方法

```java
protected HandlerExecutionChain getHandlerExecutionChain(Object handler, HttpServletRequest request) {

    String lookupPath = this.urlPathHelper.getLookupPathForRequest(request, LOOKUP_PATH);
    // 遍历拦截器，找到跟当前 url 对应的，添加到执行链中
    for (HandlerInterceptor interceptor : this.adaptedInterceptors) {
        if (interceptor instanceof MappedInterceptor) {
            MappedInterceptor mappedInterceptor = (MappedInterceptor) interceptor;
            if (mappedInterceptor.matches(lookupPath, this.pathMatcher)) {
                chain.addInterceptor(mappedInterceptor.getInterceptor());
            }
        }
        else {
            chain.addInterceptor(interceptor);
        }
    }
    return chain;
}
```

### 常用注解

**1. @Controller**

单独使用 `@Controller` 不加 `@ResponseBody` 的话一般是要返回一个视图的情况，对应于前后端不分离的情况。

和 `@ResponseBody` 配合可以返回返回 JSON 数据

> `@ResponseBody` 注解的作用是将 `Controller` 的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到 HTTP 响应 (Response) 对象的 body 中，通常用来返回 JSON 或者 XML 数据，返回 JSON 数据的情况比较多。

**2. @RestController**

`@RestController` 只返回对象，对象数据以 JSON 或 XML 的形式写入 HTTP 响应中。

**3. `@RequestParam` 以及 `@Pathvairable `**

- `@PathVariable` 用于获取路径参数
- `@RequestParam` ：如果 Controller 方法的形参和 URL 参数名一致可以不添加注解，如果不一致可以使用该注解绑定。

**4. `@RequestBody`** 

用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且 Content-Type 为 `application/json` 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用 `HttpMessageConverter` 或者自定义的 `HttpMessageConverter` 将请求的 body 中的 json 字符串转换为 java 对象。

## 事务

### 隔离级别

- `DEFAULT` 默认：使用数据库默认的事务隔离级别
- `READ UNCOMMITED`：未提交读，一个事务会**读取到另一个事务没有提交的数据**，存在脏读、不可重复读、幻读的问题。
- `READ COMMITED`：已提交读，一个事务可以**读取到另一个事务已经提交的数据**，解决了脏读的问题，存在不可重复读、幻读的问题。
- `REPEATABLE READ`：可重复读，**在一次事务中读取同一个数据结果是一样的**，解决了不可重复读的问题，存在幻读问题。
- `SERIALIZABLE`:可串行化，**每次读都需要获得表级共享锁，读写互相阻塞**，效率低，解决了幻读问题。

### 传播行为

- **REQUIRED**： 支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。
- **SUPPORTS**： 支持当前事务，如果当前没有事务，就以非事务方式执行。
- **MANDATORY**：  支持当前事务，如果当前没有事务，就抛出异常。
- **REQUIRES_NEW**： 新建事务，如果当前存在事务，把当前事务挂起。
- **NOT_SUPPORTED**： 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- **NEVER**： 以非事务方式执行，如果当前存在事务，则抛出异常。
- **NESTED**： 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与 PROPAGATION_REQUIRED 类似的操作。

# Spring Boot

### 初始化器

 Spring Boot 的系统初始化器是 SpringBoot 容器的回调接口，**会在 Spring 容器刷新前被调用**。我们可以通过系统初始化器向 Spring Boot 容器中定义属性，这个属性通常是以 K-V 的形式设置的。

- 在 `spring.factories` 文件中配置：在 `SpringApplicationContext` 构造函数中读取 `META-INF/spring.factories` 文件创建
- 在 `application.properties/yml` 文件中添加：在容器刷新前调用初始化器的 `initializer` 方法时由 `DelegatingApplicationContextInitializer` 读取配置文件添加
- 手动添加：启动前手动添加

### 自动配置

Spring Boot 可以通过 `@SpringBootApplication` 注解来实现自动注入

```java
@SpringBootConfiguration // 标识这是一个配置类
@EnableAutoConfiguration // 开启自动配置
@ComponentScan(/*包扫描，排除类*/)
public @interface SpringBootApplication {}
```

在 `@SpringBootApplication` 注解上起主要作用的有 3 个注解

- `@SpringBootConfiguration`：被 `@Configuration` 修饰，标识为一个配置类
- `@ComponentScan`：开启包扫描，默认路径为配置类路径。
- **`@EnableAutoConfiguration`**：开启自动扫描

自动配置的逻辑主要在 `@EnableAutoConfiguration` 中。

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

- `@AutoConfigurationPackage` 注解使用 `@Import` 向容器中导入了 `AutoConfigurationPackages.Registrar` 组件。**将主配置类的包名及其子包中的组件扫描到容器中**。
- `Import` 导入了 `AutoConfigurationImportSelector` 组件，获取 spring-boot-autoconfigure 下 `META-INF/spring.factories` 中配置的类，加载到容器中。

