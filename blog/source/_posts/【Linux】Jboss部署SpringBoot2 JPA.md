---
title: 【Linux】Jboss部署SpringBoot2 JPA
tags: [Linux]
date: 2020-9-18
---

# Jboss部署SpringBoot2 JPA
目录结构
```bash
.
└── webapp
    └── META-INF
        ├── jboss-deployment-structure.xml
        └── jboss-web.xml
```

**jboss-deployment-structure.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<jboss-deployment-structure>
	<deployment>
		<exclude-subsystems>

			<!-- 排除 javax.persistence.api 依赖 -->
			<subsystem name="jpa" />
                        <subsystem name="org.hibernate" />
                        <subsystem name="org.hibernate.validator" />

			<!-- 排除 org.jboss.logging 依赖 -->
			<subsystem name="logging" />
		</exclude-subsystems>
		<exclusions>
			<!--<module name="javax.validation.api" />-->
			<!--<module name="org.hibernate.validator" />-->
			<!--Log4j exclude-->
			<module name="org.slf4j" />
			<module name="org.slf4j.impl" />

			<!-- 排除 javax.persistence.api 依赖 -->
			<module name="javaee.api" />
			<module name="javax.persistence.api" />
			<module name="org.hibernate" />
                        <module name="javax.validation.api"/>
                        <module name="org.hibernate.validator"/>
		</exclusions>
		<!--<resources>
			<resource-root path="lib" />
		</resources>
		<local-last value="true" />-->
		<dependencies>

			<!-- 排除 org.apache.httpcomponents 依赖 -->
			<!-- <module name="org.apache.httpcomponents" export="true"/> -->

			<!-- 排除 org.jboss.logging 依赖 -->
			<!-- <module name="org.jboss.logging" export="true" /> -->
                        <module name="javax.validation.api" export="true"/>
			<module name="javax.activation.api" export="true" />
			<module name="javax.annotation.api" export="true" />
			<module name="javax.ejb.api" export="true" />
			<module name="javax.el.api" export="true" />
			<module name="javax.enterprise.api" export="true" />
			<module name="javax.enterprise.deploy.api" export="true" />
			<module name="javax.inject.api" export="true" />
			<module name="javax.interceptor.api" export="true" />
			<module name="javax.jms.api" export="true" />
			<module name="javax.jws.api" export="true" />
			<module name="javax.mail.api" export="true" />
			<module name="javax.management.j2ee.api" export="true" />

			<!-- 排除 javax.persistence.api 依赖 -->
			<!-- <module name="javax.persistence.api" export="true" /> -->

			<module name="javax.resource.api" export="true" />
			<module name="javax.rmi.api" export="true" />
			<module name="javax.security.auth.message.api" export="true" />
			<module name="javax.security.jacc.api" export="true" />
			<module name="javax.servlet.api" export="true" />
			<module name="javax.servlet.jsp.api" export="true" />
			<module name="javax.transaction.api" export="true" />
			<module name="javax.validation.api" export="true" />
			<module name="javax.ws.rs.api" export="true" services="export" />
			<module name="javax.xml.bind.api" export="true" />
			<module name="javax.xml.registry.api" export="true" />
			<module name="javax.xml.soap.api" export="true" />
			<module name="javax.xml.ws.api" export="true" />
			<module name="javax.api" export="true" />

			<!-- 排除掉jboss的 slf4j 和 log4j 实现，使用我们程序自己的包 -->
			<!--<module name="org.apache.log4j" export="true" />-->
			<!--<module name="org.slf4j" export="true" />-->
			<!--<module name="org.apache.commons.logging" export="true" />-->
			<module name="org.jboss.logging.jul-to-slf4j-stub" export="true" />

		</dependencies>
	</deployment>
</jboss-deployment-structure>
```

**jboss-web.xml**
```xml
<!DOCTYPE jboss-web PUBLIC "-//JBoss//DTD Web Application 5.0//EN"  
"http://www.jboss.org/j2ee/dtd/jboss-web_5_0.dtd"> 
<jboss-web>
	<context-root>app</context-root>
</jboss-web>
```


**有时启动后requestmapping无法正常加载，导致页面404，可以修改启动类如下**
```java
@SpringBootApplication
public class Application extends SpringBootServletInitializer {

    public static void main(String[] args) {
        new SpringApplicationBuilder()
                .sources(Application.class)
                .bannerMode(Banner.Mode.OFF)
                .run();
    }

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return super.configure(builder.sources(Application.class));
    }

    /**
     * 自定义DispatcherServlet,拦截/*请求
     */
    @Bean
    public ServletRegistrationBean registryRestServlet(DispatcherServlet dispatcherServlet) {
        ServletRegistrationBean<?> registrationBean = new ServletRegistrationBean<>(dispatcherServlet);
        // 这个地方是重点，注意匹配/*
        registrationBean.addUrlMappings("/*");
        return registrationBean;
    }
}
```
