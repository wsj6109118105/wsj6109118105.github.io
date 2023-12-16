---
title: spring学习笔记
date: 2021-06-09 21:41:01
tags: SSM
---
## Spring
- 总结：Spring就是一个轻量级的控制反转（IOC）和面向切面编程（AOP）的框架
## 依赖注入（DI）
- set注入
- 构造器注入
## 自动装配
1.导入约束：context约束
2.配置注解的支持：context:annotation-config/
- @Autowired:自动装配bean，可以在属性上使用，也可以在set方法上使用，使用byType方式实现
- @Qualifier(value="xxx") 如果自动配置环境比较复杂可以使用这个配合@Autowired指定唯一一个bean对象注入
- @Resource 也是用来自动装配的，默认通过byName，找不到使用byType,还找不到就报错
- @Nullable 标记这个注解说明字段可以为空

## 注解
##### 纯java配置不适用xml时要使用这些注解取实现配置类
- @Configuration代表这是一个配置类，如果使用配置类的方法，只能通过AnnotationConfig上下文类获取容器，通过配置类的class对象加载
- @bean注册一个bean，相当于一个bean标签方法的名字就相当于bean的id,方法的返回值就相当于bean的class属性
- @ComponentScan包扫描的注解
##### 正常的配置，使用xml来实现
- @Component 实现属性的注入
- @Value（"xxx"）相当于<property name="name">
- @Repository dao层使用的属性注入注解(与component功能一致)
- @Service service层使用的属性注入注解(与component功能一致)
- @Controller controller层使用的属性注入注解(与component功能一致)
- @Scope("prototype(原型模式)") 作用域的注解，默认为单例
- @Aspect标注这个类是一个切面
- @Before/After/Around("execution(切入点)")
## 代理模式
**代理模式是SpringAOP的底层**
#### 代理模式分类：
- 静态代理
- 动态代理
##### 静态代理
角色分析
- 抽象角色：一般会使用接口或者抽象类来解决
- 真实角色：被代理的角色
- 代理角色：代理真实角色，代理真实角色后，我们一般会做一些附属操作
- 客户：访问代理对象的人！
代码步骤：
1.接口
2.真实角色
3.代理角色
4.客户端访问代理角色
代理模式的好处：
- 可以是真实角色的操作更加纯粹！不用取关注一些公共的业务
- 公共业务就交给了代理角色！实现了业务的分工
- 公共业务发生扩展的时候，方便集中管理
缺点
- 一个真实角色就会产生一个代理角色；代码量会翻倍；开发效率会变低
##### 动态代理
**动态代理和静态代理角色一样**
**动态代理类是动态生成的，不是我们直接写好的**
动态代理分为两大类：
- 基于接口的动态代理：JDK动态代理
- 基于类的动态代理：cglib
**java字节码：javasist**
**需要了解两个类：Proxy:代理,InvocationHandler:调用处理程序**
##### 生成代理类
- 实现InvocationHandler接口，重写invoke方法
#### 使用Spring实现AOP
AOP织入aspectjweaver
##### 使用Spring的API接口(主要时SpringAPI接口实现)
- MethodBeforeAdvice接口 前置
- MethodAfterAdvice接口 后置
参数：
- method:要执行的目标对象的方法
- args:参数
- o:目标对象
之后配置aop
```xml
<aop:config>
    <!--配置切入点(可以配置多个)-->
    <aop:pointcut id="pointcut" expression(* com.service.UserServiceImpl.*(..))/>
    <!--执行环绕增加-->
    <aop:advisor advice-ref="log" pointcut-ref="pointcut"/>
    <aop:advisor advice-ref="afterLog" pointcut-ref="pointcut"/>
</aop:config>
```
##### 使用自定义类来实现(主要是切面定义)
- 自定义一个类
- 配置<aop:config>配置切面<zop:aspect ref="diy">ref要引用的类；之后配置切入点，切面。
## 声明式事务
### 1.回顾事务
- 把一组业务当成一个来做要么都成功，要么都失败
- 确保完整性和一致性 
事务的ACID原则
- 原子性
- 一致性
- 隔离性
- 持久性
### 2.spring中的事务管理
- 声明式事务
配置声明式事务
结合AOP实现事务的织入
配置事务通知