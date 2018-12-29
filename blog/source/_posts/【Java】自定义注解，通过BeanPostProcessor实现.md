---
title: 【Java】自定义注解，通过BeanPostProcessor实现
tags: [java]
date: 2018-12-6
---

## springboot中实现自定义注解
#### annotation 注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD})
@Documented
@Inherited
public @interface Log {
    String description() default "";
}
```
#### aspect 解释器
 ```java
@Component
@Aspect
public class LogAspect implements BeanPostProcessor {
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
     throws BeansException {
        ReflectionUtils.doWithFields(bean.getClass(), field -> {
            ReflectionUtils.makeAccessible(field);
            // 查找是否有Log.class类型的注解
            if (null != field.getAnnotation(Log.class)){
                final Logger log = LoggerFactory.getLogger(bean.getClass());
                field.set(bean,log);
            }
        });
        return bean;
    }
}
 ```