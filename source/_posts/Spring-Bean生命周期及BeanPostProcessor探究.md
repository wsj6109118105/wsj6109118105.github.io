---
title: Spring Bean生命周期及BeanPostProcessor探究
date: 2021-10-01 11:07:04
tags: Spring
---
## Spring Bean生命周期测试
控制台输出顺序：
```bash
setBeanName被调用了---->BeanNameAware接口方法

setBeanFactory被调用了---->BeanFactoryAware接口方法

setApplicationContext被调用了---->ApplicationContextAware接口方法

afterPropertiesSet被调用了---->InitializingBean接口方法

postProcessBeforeInitialization被调用了---->BeanPostProcessor接口方法

postProcessAfterInitialization被调用了---->BeanPostProcessor接口方法

destroy被调用了---->DisposableBean接口方法
```
由此可知Bean生命周期中实例化，依赖注入之后的执行顺序

> 通过@Bean定义初始化和销毁方法

测试类People

控制台输出：
```bash
People无参构造
初始化
销毁
```
> 通过@PostConstruct和@PreDestroy定义初始化和销毁方法

测试类Cat

控制台输出：
```bash
无参构造猫
猫出生
猫老去
```

> 测试 BeanPostProcessor 接口( Bean后置通知处理器)

> 第一次在 People 上实现接口

共有3个Bean：People,Cat,Dog

```java
public class People implements BeanPostProcessor{
    public People() {
        System.out.println("People无参构造");
    }

    public void init() {
        System.out.println("初始化");
    }

    public void destory(){
        System.out.println("销毁");
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "初始化调用之前");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "初始化调用之后");
        return bean;
    }
}
```
控制台输出：
```bash
People无参构造
初始化
无参构造猫
cat初始化调用之前
猫出生
cat初始化调用之后
狗无参构造
dog初始化调用之前
Dog born
dog初始化调用之后
Dog die
猫老去
销毁
```
结果并没有对People生效，从输出中发现 People 为最先创建的Bean,之后是 lifeCircleConfig 然后出现一条信息表明 lifeCircleConfig 创建在了 BeanPostProcessors 之前，所以才会只对 Cat,Dog 生效
```bash
21:52:41.124 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'people'

21:52:41.128 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'lifeCircleConfig'

21:52:41.138 [main] INFO org.springframework.context.support.PostProcessorRegistrationDelegate$BeanPostProcessorChecker - Bean 'lifeCircleConfig' of type [beanlifecircle.config.LifeCircleConfig$$EnhancerBySpringCGLIB$$a3479ebb] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
```

> 第二次测试，单独实现 BeanPostProcessor
```java
public class MyBeanProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "初始化调用之前");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "初始化调用之后");
        return bean;
    }
}
```
控制台输出：
```bash
People无参构造
people初始化调用之前
初始化
people初始化调用之后
无参构造猫
cat初始化调用之前
猫出生
cat初始化调用之后
狗无参构造
dog初始化调用之前
Dog born
dog初始化调用之后
Dog die
猫老去
销毁
```
可见对除了本身和 lifeCircleConfig 之外的 Bean 全部生效了

总结：**BeanPostProcessor 本身也是一个 Bean 而且这个接口是对全体 Bean 生效的，在使用时应当独立实现，而不应该在已有的 Bean 上实现，否则对实现的Bean本身是不起作用的**

> 补充

**由于BeanPostProcessor原因，最开始各个方法的执行顺序是有误的，因为在Bean本身实现了BeanPostProcessor**

去掉后测试
```bash
setBeanName被调用了---->BeanNameAware接口方法

setBeanFactory被调用了---->BeanFactoryAware接口方法

setApplicationContext被调用了---->ApplicationContextAware接口方法

beanLifeCircle初始化调用之前---->BeanPostProcessor接口方法

afterPropertiesSet被调用了---->InitializingBean接口方法

beanLifeCircle初始化调用之后---->BeanPostProcessor接口方法

destroy被调用了---->DisposableBean接口方法
```

[源码](https://github.com/wsj6109118105/spring/tree/main/BeanLifeCircle)展示