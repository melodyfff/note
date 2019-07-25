---
title: 【Java】使用JAVA校验XML Schema
tags: [java]
date: 2019-7-25
---

# 使用JAVA校验XML Schema

近期由于工作上的需求，需要对请求的`xml`进行`xsd`校验

于是想到了如下的方法：

所有的的`.xsd`校验文件都放在`resources/xsd`目录下


## 遇到的问题

### 国际化
首先是需要返回的提示信息要国际化

该类国际化资源的信息存放在`jdk`的`jre/lib/resources.jar`文件中的`com/sun/org/apache/xerces/internal/impl/msg`路径下

需要设置为何种语言只需获取到`javax.xml.validation.Validator`后，通过如下代码设置语言环境即可

```java
validator.setProperty("http://apache.org/xml/properties/locale", Locale.SIMPLIFIED_CHINESE);
```

### 部署到jboss上xsd文件路径获取不正确

由于项目要部署到`jboss`上，`jboss`上的文件系统是`vfs`的，所以当执行以下代码时，会产生错误：

```java
// 尝试在classpath路径下（即编译后的target/classes）寻找该文件
xsdFile = ResourceUtils.getFile("classpath:" + xsdPath);
```

为了能获取到正确的路径，得借助`jboss-vfs`的`org.jboss.vfs.VirtualFile`

```java
      // 由于jboss上使用vfs文件，可能查询不到路径，得借助jboss-vfs获取真实路径
      if (null== xsdFile){
        final VirtualFile content = (VirtualFile) XmlValidateUtil.class.getClassLoader().getResource(xsdPath).getContent();
        log.debug("校验规则 {} 真实路径为 ：{}",xsdPath,content.getPhysicalFile().getPath());
        xsdFile = content.getPhysicalFile();
      }
```

### jboss上设置国际化失败

这个文件主要是因为`jboss`启动时，自己的类加载器加载了`apache`的`xerces`，`jdk`中使用的是`com/sun/org/apache/xerces`

这样就导致我们获取到的`Validator`是以`apache`中的相关部分来实现的，不能直接使用`jdk`中默认的

为了能使用`jdk`自带的部分，只需获取到该类加载器的父类加载器去加载即可（绕过jboss的类加载器）

并且在生成`javax.xml.validation.SchemaFactory`时，使用`AppClassLoader`去加载相关的部分即可

```java
// jboss和tomcat中类加载器不是常规的双亲委派，通过getParent（）获取AppClassLoader
      ClassLoader classLoader = XmlValidateUtil.class.getClassLoader();
      if (!(classLoader instanceof URLClassLoader)){
        classLoader = classLoader.getParent();
      }
      // 指定使用jdk自带的XMLSchemaFactory，为了使用国际化资源
      SchemaFactory schemaFactory =  SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema","com.sun.org.apache.xerces.internal.jaxp.validation.XMLSchemaFactory",classLoader);
```

--- 

## 完整工具类文件

**XmlValidateUtil.java**

```java
@Slf4j
public final class XmlValidateUtil {

  public static final  String XSD1 = "xsd/XSD1.xsd";
  public static final  String XSD2 = "xsd/XSD2.xsd";

  static Map<String, Validator> validatorMap = new HashMap<>();

  static {
    for (String key :of(XSD1, XSD2)) {
      try {
        validatorMap.put(key, getValidator(key));
      } catch (Exception e) {
        log.error("init xsd map fail.");
      }
    }
  }


  private XmlValidateUtil() {
  }


  public static void validateXmlByXsd(String xml, String xsdPath) throws Exception {
    try {
      Source xmlFile = new StreamSource(new StringReader(xml));
      Validator validator = validatorMap.get(xsdPath);
      validator.validate(xmlFile);
      log.info("xsd valid success.");
    }  catch (SAXException var8) {
      log.error("xsd valid error.");
      log.error("Reason: {}", var8.getLocalizedMessage());
      throw new Exception(var8.getLocalizedMessage());
    } catch (Exception e){
      log.error("获取校验规则 【{}】 路径失败 : {}",xsdPath,e);
    }
  }


  static Validator getValidator(String xsdPath) throws Exception {
    File xsdFile = null;
    try {
      // 通过类加载器获取真实路径
      xsdFile = ResourceUtils.getFile("classpath:" + xsdPath);
    } catch (Exception e){
      log.debug("----" + (new ClassPathResource(xsdPath)).getURI() + "");
    }

    try {
      // 由于jboss上使用vfs文件，可能查询不到路径，得借助jboss-vfs获取真实路径
      if (null== xsdFile){
        final VirtualFile content = (VirtualFile) XmlValidateUtil.class.getClassLoader().getResource(xsdPath).getContent();
        log.debug("校验规则 {} 真实路径为 ：{}",xsdPath,content.getPhysicalFile().getPath());
        xsdFile = content.getPhysicalFile();
      }

      // jboss和tomcat中类加载器不是常规的双亲委派，通过getParent（）获取AppClassLoader
      ClassLoader classLoader = XmlValidateUtil.class.getClassLoader();
      if (!(classLoader instanceof URLClassLoader)){
        classLoader = classLoader.getParent();
      }
      // 指定使用jdk自带的XMLSchemaFactory，为了使用国际化资源
      SchemaFactory schemaFactory =  SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema","com.sun.org.apache.xerces.internal.jaxp.validation.XMLSchemaFactory",classLoader);
      Schema schema = schemaFactory.newSchema(xsdFile);
      Validator validator = schema.newValidator();
      // 尝试设置异常的语言环境
      // 使用jdk中的
      validator.setProperty("http://apache.org/xml/properties/locale", Locale.SIMPLIFIED_CHINESE);
      return validator;
    } catch (Exception e){
      log.error("获取校验规则 【{}】 路径失败 : {}",xsdPath,e);
      throw new Exception("获取校验规则失败");
    }
  }

  static <T> T[] of(T... values){
    return values;
  }
}
```