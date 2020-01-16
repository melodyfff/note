---
title: 【Spring】Spring 通配符加载Resource文件
tags: [spring]
date: 2020-1-16
---

# Spring 通配符加载Resource文件

`Spring`中`Resource`继承自`InputStreamSource`

其中`InputStreamSource`接口定义了获取`java.io.InputStream`流的规范

**InputStreamSource**
```java
public interface InputStreamSource {
	/**
	 * 返回{@link InputStream}
	 * 每次调用都创建新的steram
	 * @return the input stream for the underlying resource (必须不为{@code null})
	 * @throws java.io.FileNotFoundException 如果resource不存在抛出异常
	 * @throws IOException 如果无法打开抛出IO异常
	 */
	InputStream getInputStream() throws IOException;
}
```

对于**Resource**定义了操作资源文件的一些基本规范
```java
public interface Resource extends InputStreamSource {
	boolean exists();
	default boolean isReadable() {
		return exists();
	}
	default boolean isOpen() {
		return false;
	}
	default boolean isFile() {
		return false;
	}
	URL getURL() throws IOException;
	URI getURI() throws IOException;
	File getFile() throws IOException;
	default ReadableByteChannel readableChannel() throws IOException {
		return Channels.newChannel(getInputStream());
	}
	long contentLength() throws IOException;
	long lastModified() throws IOException;
	Resource createRelative(String relativePath) throws IOException;
	@Nullable
	String getFilename();
	String getDescription();
}
```

Spring中加载资源文件主要是`ResourceLoader`接口,里面定义了获取资源以及类加载器的规范
**ResourceLoader**
```java
public interface ResourceLoader {
    /** Pseudo URL prefix for loading from the class path: "classpath:". */
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;

    Resource getResource(String location);

    ClassLoader getClassLoader();
}
```

Spring中的所有资源加载都是在该接口的基础上实现的

对于通配符加载多个`Resource`文件的主要是
**ResourcePatternResolver**
```java
public interface ResourcePatternResolver extends ResourceLoader {
	String CLASSPATH_ALL_URL_PREFIX = "classpath*:";

	Resource[] getResources(String locationPattern) throws IOException;
}
```

通过`getResources()`方法即可匹配多个

比较常用的实现类为`PathMatchingResourcePatternResolver`

以下为配置`mybatis`的`SqlSessionFactoryBean`时,设置`MapperLocations`的使用
```java
    @Bean
    public SqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) throws IOException {
        final SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        final PathMatchingResourcePatternResolver pathMatchingResourcePatternResolver = new PathMatchingResourcePatternResolver();
        sqlSessionFactoryBean.setMapperLocations(pathMatchingResourcePatternResolver.getResources("classpath:mapper/*.xml"));
        return sqlSessionFactoryBean;
    }
```