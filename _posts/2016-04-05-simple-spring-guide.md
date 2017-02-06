---
layout: post
title: "Simple spring guide"
category: NOTE
tags: [Java, SPRING]
---
* TOC
{:toc}
# Spring
> 核心 IOC、Bean、AOP

## IOC & Bean
> 控制反转，DI依赖注入而不是主动new，管理着Bean的生命周期
> 在Spring中一切皆Bean，IOC容器实质包含的都是Bean，可以简单的理解为BeanFactory

Bean的Scope
1. Singleton，单例默认作用域
2. Prototype，每次请求都创建新的对象
3. Request，每次http请求会创建新的对象，request唯一
4. Session，Session中唯一
5. Global session

Bean的生命周期
```java
// 初始化 接口优先于配置
public class BeanInit implements InitializingBean {
    @Override
    public void afterPropertiesSet() ...{
        // TODO
    }
}
// 或者在配置文件中添加属性init-method="init"

// 销毁 接口优先于配置
// 实现DisposableBean接口或者添加destroy-method="destroy"
// 全局属性 default-init-method or default-destroy-method 可覆盖
```
Aware接口（获取Spring对象）
ApplicationContextAware，BeanNameAware

Bean Autowire
1. ByName，根据属性名（ID唯一）
2. ByType，根据属性类型
3. Constructor，用于构造器参数（ByType）

Resources
1. UrlResource，URL对应的资源，根据一个URL地址即可构建
2. ClassPathResource，获取类路径下的资源文件（相对地址）
3. FileSystemResource，获取文件系统里面的资源（绝对地址）
4. ServletContextResource，访问ServletContext中的资源
5. InputStreamResource，针对于输入流封装的资源
6. ByteArrayResource，针对于字节数组封装的资源

ResourceLoader
> All application context implements resource loader interface and therefore all application context may be used to obtain resource

Prefix
1. classpath:，类路径
2. file:，文件系统
3. url:，URL
4. none，当前ApplicationContext的方式

> Since 3.0提供了注解方式初始化IOC

1. @Component是一个通用注解，可用于任何Bean
    @Repository用于Dao持久层
    @Service用于服务层（逻辑层）
    @Controller用于控制层（MVC）
    ```java
        @Target({ElementType.TYPE})
        @Retention(RetentionPolicy.RUNTIME)
        @Documented
        @Component    //meta-annotation, Service implements Component
        public @interface Service {
            // TODO
        }
    ```
    ```xml
        <context:annotation-config/> 仅会查找在同一个ApplicationContext中的Bean注解
        <context:component-scan base-package="com.codeyn"/>该标签包含annotation-config，常用component-scan，也包 含AutowireAnnotationBeanPostProcessor和CommonAnnotationBeanPostProcessor
        // 使用Filter自定义扫描
        <context:component-scen base-package="com.codeyn">
            <context:include-filter type="regex" expression=".*Stub.*Repository"/>
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
        </context:component-scan>
        // 使用use-default-filter = "false"禁止自动扫描
    ```

Annotation Filter Type
1. annotation
2. assignable
3. aspectj
4. regex
5. custom

BeanNameGenerator生成Bean Name，默认类名首字母小写，或者在注解value中指定，或者实现BeanNameGenerator接口（需要包含一个无参构造器）
```xml
    <context:component-scan base-package="com.codeyn" name-generator="com.codeyn.XXGenerator"/>
```
作用域（scope("prototype")），可以实现ScopeMetadataResolver接口（无参构造器）
```xml
    <context:component-scan base-package="com.codeyn" scope-resolver="..ScopeResolver"/>
```
代理方式（scope-proxy）
1. no（默认）
2. interfaces
3. targetClass

```xml
    <context:component-scan base-package="com.codeyn" scoped-proxy="interfaces"/>
```
自动装配
> @Autowired是由BeanPostProcessor处理，通过xml或@Bean加载

1. Required（setter）
2. Autowired（field、setter）
    Set、List、Map
    Spring Bean
    Bean实现Ordered | @Order

@Qualifier（多类型值缩小范围）
1. field、constructor、parameter

@Resource（名称）
1. 名称由CommonAnnotationBeanPostProcessor提供
2. 如果CommonAnnotationBeanPostProcessor在ApplicationContext中注册可以使用@PostConstruct & @PreDestroy初始化和销毁

@Bean（类型<bean/>，与@Configuration结合使用）

```xml
<context:property-placeholder location="classpath:/com/codeyn/datasource.properties"/>
<bean class="x">
    <property name="url" value="${jdbc.url}"/>
</bean>
```
```java
@Configuration
@ImportResource("classpath:/com/codeyn/x.xml")
public class XConfig {
    @Value("${jdbc.url}")
    private String url;
}
```
泛型（4.x）
CustomAutowireConfigurer（BeanFactoryPostProcessor的之类，可以注册自己的@Qualifier）
```xml
<bean id="" class="">
    <property name="customQualifierType">
        <set><value>com.codeyn.CustomQualifier</value></set>
    </property>
</bean>
autowire-candidate
default-autowire-candidates
```
JSR330注解
```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```
@Inject等效于@Autowired（field、method、class、constructor）
@Named等效于@Component

## AOP
相关概念
1. Aspect，一个关注点的模块化，可能会横切多个obj
2. Joinpoint，执行中的某个特定点
3. Advice，在切面的Joinpoint上执行的动作
4. Pointcut，匹配连接点的断言，在AOP中通知和一个切入点表达式关联
5. Introduction，在不修改代码的前提下添加新方法和属性
6. Target，被Aspect通知的obj
7. AOP proxy，AOP框架创建的对象，用来实现Aspect contract
8. Weaving，把aspect连接到其他的程序类型中，并创建一个被通知的obj（编译时、类加载时、执行时）

### Based on schema aspect
```xml
<aop:config>
    <aop:aspect id="aspect1" ref="bean1">
        <aop:pointcut id="xx" expression="execution(* com.codeyn.aop.schema.advice.*.*(..)) and args(...)" />
        <aop:before pointcut-ref="xx" method="before" />
        <aop:after-returning method="afterReturning" pointcut-ref="xx" returning="retVal" />
        <aop:after-throwing pointcut-ref="xx" method="afterThrowing" />
        <aop:after-throwing pointcut-ref="xx" throwing="xxEx" method="xAdvice" />
        <aop:after pointcut-ref="xx" method="xAfter" />
        <aop:around pointcut-ref="xx" method="xAround" />
    </aop:config>
</aop:config>
<bean id="bean1" class="xAop">
</bean>
```

