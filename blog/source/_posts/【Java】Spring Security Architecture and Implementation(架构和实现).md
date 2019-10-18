---
title: 【Java】Spring Security Architecture and Implementation(架构和实现)
tags: [java]
date: 2019-10-18
---

# Spring Security Architecture and Implementation(架构和实现)

关于`spring-security`的官网文档学习笔记,主要是`第8章 Architecture and Implementation(架构和实现)`内容

参考: https://docs.spring.io/spring-security/site/docs/5.2.1.BUILD-SNAPSHOT/reference/htmlsingle/#overall-architecture

## Architecture and Implementation(架构与实现)
应用程序安全性可以归结为或多或少的两个独立问题：身份验证（您是谁）和授权（您可以做什么？）

---

### Summary(摘要)
`Spring Security`的主要组成部分:
- `SecurityContextHolder`，提供对的访问`SecurityContext`。
- `SecurityContext`，以保存`Authentication`（可能还有特定于请求的）安全信息。
- `Authentication`，以特定于`Spring Security`的方式表示主体。
- `GrantedAuthority`，以反映授予主体的应用程序范围的权限。
- `UserDetails`，以提供必要的信息以从应用程序的DAO或其他安全数据源构建`Authentication`对象。
- `UserDetailsService`，以`UserDetails`在传入String基于-的用户名（或证书ID等）时创建一个。

---

### Authentication(身份验证)

#### What is authentication in Spring Security?
`Spring Security`中的身份验证:
- 获取用户名和密码，并将其组合到的一个实例`UsernamePasswordAuthenticationToken`（`Authentication`接口的实例）。
- 令牌会传递到的实例`AuthenticationManager`进行验证。
- 成功认证后 ，`AuthenticationManager`返回一个完全填充的`Authentication`实例。
- 通过调用`SecurityContextHolder.getContext().setAuthentication(…​)`并传入返回的身份验证对象来建立安全上下文。

**官网实例:**
```java
/**
 *
 * 身份认证实例
 *
 * {@link AuthenticationManager} 认证管理器,处理{@link Authentication}请求,返回一个完全填充的{@link Authentication}实例
 *
 * {@link GrantedAuthority} 授予的权限
 *
 * 参考: https://docs.spring.io/spring-security/site/docs/5.2.1.BUILD-SNAPSHOT/reference/htmlsingle/#tech-intro-authentication
 *
 * @author xinchen
 * @version 1.0
 * @date 16/10/2019 11:07
 */
public class AuthenticationExample {
    private static AuthenticationManager am = new SampleAuthenticationManager();

    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));

        while (true){

            System.out.println("Please enter your username:");
            String name = in.readLine();
            System.out.println("Please enter your password:");
            String password = in.readLine();

            try {
                Authentication request = new UsernamePasswordAuthenticationToken(name, password);
                Authentication result = am.authenticate(request);
                SecurityContextHolder.getContext().setAuthentication(result);
                break;
            } catch(AuthenticationException e) {
                System.out.println("Authentication failed: " + e.getMessage());
            }
        }

        System.out.println("Successfully authenticated. Security context contains: " +
                SecurityContextHolder.getContext().getAuthentication());
    }
}

class SampleAuthenticationManager implements AuthenticationManager{

    static final List<GrantedAuthority> AUTHORITIES = new ArrayList<>();

    static {
        // 初始化角色
        AUTHORITIES.add(new SimpleGrantedAuthority("ROLE_USER"));
    }

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        // authentication.getCredentials() 通常是密码
        if (authentication.getName().equals(authentication.getCredentials())){
            // Object principal, Object credentials, Collection<? extends GrantedAuthority> authorities
            return new UsernamePasswordAuthenticationToken(authentication.getName(), authentication.getCredentials(), AUTHORITIES);
        }
        throw new BadCredentialsException("Bad Credentials");
    }
}
```

---

### Authentication in a Web Application
考虑典型的`Web`应用程序的身份验证过程：
- 1.您访问主页，然后单击链接。
- 2.请求发送到服务器，服务器确定您已请求受保护的资源。
- 3.由于您目前尚未通过身份验证，因此服务器会发回响应，指示您必须进行身份验证。响应将是HTTP响应代码，或重定向到特定网页。
- 4.根据身份验证机制，您的浏览器将重定向到特定网页，以便您可以填写表格，或者浏览器将以某种方式检索您的身份（通过BASIC身份验证对话框，cookie，X.509证书等）。 ）。
- 5.浏览器将响应发送回服务器。这将是包含您填写的表单内容的HTTP POST或包含身份验证详细信息的HTTP标头。
- 6.接下来，服务器将决定所提供的凭据是否有效。如果有效，则将进行下一步。如果它们无效，通常会要求您的浏览器再试一次（因此您返回到上面的第二步）。
- 7.您尝试引起身份验证过程的原始请求将被重试。希望您已获得足够授权的身份验证，以访问受保护的资源。如果您具有足够的访问权限，则请求将成功。否则，您将收到一个HTTP错误代码403，表示“禁止”。

`Spring Security`具有负责上述大多数步骤的不同类。
主要参与者（按照使用顺序）是`ExceptionTranslationFilter`，`AuthenticationEntryPoint`和和`“Authentication Mechanism(身份验证机制)”`，它们负责调用`AuthenticationManager`

#### ExceptionTranslationFilter
`ExceptionTranslationFilter`是一个Spring Security过滤器，它负责检测抛出的任何Spring Security异常(`AccessDeniedException`和`AuthenticationException`).
此类异常通常由`AbstractSecurityInterceptor`授权服务的主要提供者抛出

#### AuthenticationEntryPoint
`AuthenticationEntryPoint(认证入口点)`主要负责上述步骤中的第`3`步.
主要是在`ExceptionTranslationFilter`中通过捕获到`AuthenticationException`异常后,
调用`handleSpringSecurityException(...)`在此方法中的`sendStartAuthentication(...)`开启认证方案

#### Authentication Mechanism
`Authentication Mechanism(身份验证机制)`,
一旦您的浏览器提交了身份验证凭据(authentication credentials)（作为HTTP表单帖子或HTTP标头
,服务器上就需要有一些东西来“收集”这些身份验证详细信息。到目前为止，我们位于上述列表的第`6`步.
在`Spring Security`中，我们有一个特殊的名称，用于从用户代理（通常是Web浏览器）收集身份验证详细信息的功能，将其称为`Authentication Mechanism(身份验证机制)`。
从用户代理收集到身份验证详细信息后，`Authentication`便会构建一个`“请求”`对象，然后将其呈现给`AuthenticationManager`

`Authentication Mechanism(身份验证机制)`收到完整的`Authentication(从AuthenticationManager中返回)`对象后，
它将认为请求有效，将`Authentication`放入`SecurityContextHolder`，
并导致重试原始请求（上述步骤`7`）。
另一方面，如果`AuthenticationManager`拒绝了请求，
则认证机制将要求用户代理重试（上面的第`2`步）。

通常`AuthenticationManager`中的返回:
- 如果它可以验证输入是否代表有效的主体，则返回`Authentication`（通常为`authenticated=true`）
- 如果它认为输入代表无效的主体，则抛出一个`AuthenticationException`
- 如果无法决定，则返回 `null`

#### Storing the SecurityContext between requests
在请求之间如何存储`SecurityContext`?根据应用程序的类型，可能需要制定一种策略来存储用户操作之间的安全上下文。在典型的Web应用程序中，用户登录一次，然后通过其会话ID(`session id`)进行标识。
有效期取决于服务器`session`的过期时间.

在`Spring Security`中，存储`SecurityContext`请求之间的责任落在`SecurityContextPersistenceFilter`，
默认情况下，将上下文存储为`HttpSession`HTTP请求之间的属性(将设置`request.setAttribute(FILTER_APPLIED, Boolean.TRUE);`保证值执行一次)。
它将上下文`SecurityContextHolder`还原到每个请求，并且至关重要的是，当请求完成时清除`SecurityContextHolder`


`SecurityContextPersistenceFilter`中真正存储由`SecurityContextRepository`完成,通过调用`loadContext(...)加载`其中:
- `NullSecurityContextRepository`实际上是每次都生成新的`SecurityContextHolder.createEmptyContext()`
- `HttpSessionSecurityContextRepository`实际上是存储在`HttpSession`中,默认属性值为`SPRING_SECURITY_CONTEXT`,可以通过修改`springSecurityContextKey`自定义该值

许多其他类型的应用程序（例如，无状态RESTful Web服务）不使用HTTP会话，并且将在每个请求上重新进行身份验证。
但是，仍然必须确保链中包含`SecurityContextPersistenceFilter`，以确保`SecurityContextHolder`在每次请求后清除。

---

### Access-Control (Authorization) in Spring Security
在`Spring Security`中负责做出访问控制决策的主要接口是`AccessDecisionManager`。它具有一种`decide`方法，该方法采用`Authentication`代表请求访问的主体的对象，`“secure object(安全对象)”`（请参见下文）以及适用于该对象的安全元数据属性列表（例如授予访问权限所需的角色列表） ）。

#### Security and AOP Advice
您可以选择使用`AspectJ`或`Spring AOP`执行方法授权，也可以选择使用过滤器执行Web请求授权,`Spring Security`中主要是`AbstractSecurityInterceptor`的继承类`FilterSecurityInterceptor`贯穿整个周期

#### Secure Objects and the AbstractSecurityInterceptor
那么，什么是 `“secure object(安全对象)”`？`Spring Security`使用该术语来指代任何可以对其应用安全性（例如授权决策）的对象。最常见的示例是方法调用和Web请求。

每个受支持的安全对象类型都有其自己的拦截器类(`AbstractSecurityInterceptor`的子类),
重要的是,当`AbstractSecurityInterceptor`被调用时,如果`principal(主体)`已通过身份验证，
`SecurityContextHolder`则将包含有效`Authentication`的内容。

`AbstractSecurityInterceptor` 提供用于处理安全对象请求的一致工作流，通常是：

- 1.查找与当前请求关联的`“configuration attributes(配置属性)”`
- 2.将安全对象，当前`Authentication`属性和配置属性提交`AccessDecisionManager`给授权决策
- 3.（可选）更改`Authentication`调用的依据
- 4.允许进行安全对象调用（假设已授予访问权限）
- 5.调用返回后如果配置了`AfterInvocationManager`,则调用。一旦调用引发了异常，`AfterInvocationManager`则不会调用。

> 注,关于步骤`2`,可查看`AbstractSecurityInterceptor`中的`this.accessDecisionManager.decide(authenticated, object, attributes);`

##### What are Configuration Attributes(配置属性)?
可以将`“configuration attribute(配置属性)”`视为一个`String`，它对所有`AbstractSecurityInterceptor`使用的类具有特殊的含义.
它们由框架内的接口`ConfigAttribute`表示。它们可以是简单的角色名称，也可以具有更复杂的含义，具体取决于`AccessDecisionManager`的具体实现。
`AbstractSecurityInterceptor`配置了`SecurityMetadataSource`来查找安全对象的属性。通常，此配置将对用户隐藏。
配置属性将作为安全方法的注释或安全URL的访问属性输入.
例如，当我们配置`<intercept-url pattern='/secure/**' access='ROLE_A,ROLE_B'/>`，这表示配置属性`ROLE_A`和`ROLE_B`适用于与给定模式匹配的网络请求。
实际上，使用默认`AccessDecisionManager`配置,这意味着`GrantedAuthority`将允许具有这两个属性之一匹配的任何人访问。
严格来说，它们只是属性，其解释取决于`AccessDecisionManager`的实现。
前缀的使用`ROLE_`是一个标记，以指示这些属性是角色，并且应由`Spring Security`的`RoleVoter`消费.
这仅`AccessDecisionManager`在使用基于投票者的情况下才有意义。

##### RunAsManager
假设`AccessDecisionManager`决定允许该请求,`AbstractSecurityInterceptor`通常将继续进行该请求。
但是有可能用户希望用不同的`Authentication`替换处于`SecurityContext`中已经通过`AccessDecisionManager`处理返回的`Authentication`,这个时候可以使用`RunAsManager`.
当服务层方法(Service layer)需要调用远程系统并显示不同的标识(不同的安全属性等)时,这可能会有有用.
因为`Spring Security`会自动将安全身份从一台服务器传播到另一台服务器（假设您使用的是正确配置的`RMI`或`HttpInvoker`远程协议客户端）

##### AfterInvocationManager
当安全对象(secure object)调用和返回,这可能意味着方法调用完成或过滤器链继续进行,`AbstractSecurityInterceptor`最终有机会处理该调用。
在这个阶段,`AbstractSecurityInterceptor`可以修改返回对象.
我们可能希望发生这种情况，因为无法在安全对象调用的“途中”做出授权决定。
由于高度可插拔，因此`AbstractSecurityInterceptor`将控制权传递给`AfterInvocationManager`以便在需要时实际修改对象。
此类甚至可以完全替换对象，或者引发异常，也可以按照其选择的任何方式对其进行更改。
调用后检查仅在调用成功的情况下执行。如果发生异常，将跳过其他检查。

**Security interceptors and the "secure object" model**
![](../img/security-interception.png)

##### Extending the Secure Object Model
只有打算采用全新的拦截和授权请求方式的开发人员才需要直接使用安全对象。
例如，有可能建立一个新的安全对象以保护对消息系统的呼叫。
任何需要安全性并且还提供拦截呼叫的方式的东西（例如，围绕建议语义的AOP）都可以成为安全对象。
话虽如此，大多数Spring应用程序将完全透明地使用当前支持的三种安全对象类型（AOP Alliance `MethodInvocation`，AspectJ `JoinPoint`和Web request `FilterInvocation`）。

---

### Core Services(核心服务)
现在，我们对`Spring Security`的架构和核心类的高度概括，让我们来仔细看看一个或两个核心接口及其实现的，
特别是`AuthenticationManager`，`UserDetailsService`和`AccessDecisionManager`。

#### AuthenticationManager，ProviderManager和AuthenticationProvider
![](../img/authentication.png)
`AuthenticationManager`只是一个接口，这样实现可以让我们选择，但它是如何在实践中运作？
如果我们需要检查多个身份验证数据库或不同身份验证服务（例如数据库和LDAP服务器）的组合，该怎么办？

`Spring Security`中的默认实现是`ProviderManager`,而不是处理身份验证请求本身,
它委托给已配置的`AuthenticationProviders`列表，依次查询每个，以查看其是否可以执行身份验证。
每个provider都会引发异常或者返回一个完全填充的`Authentication`对象.
验证身份验证请求的最常见方法是加载相应`UserDetails`的密码，并对照用户输入的密码检查已加载的密码。
这是`DaoAuthenticationProvider`（见下文）使用的方法。加载的`UserDetails`对象-尤其是`GrantedAuthority`包含的对象-
当成功认证时,会构建一个完全填充的`Authentication`对象返回,并且存储到`SecurityContext`环境上下文中.

```xml
<bean id="authenticationManager"
        class="org.springframework.security.authentication.ProviderManager">
    <constructor-arg>
        <list>
            <!--     这些bean都是AuthenticationProvider的实现类    -->
            <ref local="daoAuthenticationProvider"/>
            <ref local="anonymousAuthenticationProvider"/>
            <ref local="ldapAuthenticationProvider"/>
        </list>
    </constructor-arg>
</bean>
```

在上面的示例中，我们有三个提供程序。它们按照所示的顺序进行尝试.
每个提供程序都可以尝试进行身份验证，也可以通过简单地返回来跳过身份验证`null`。
如果所有实现均返回`null`，`ProviderManager`则将抛出`ProviderNotFoundException`。

诸如Web表单登录处理过滤器之类的身份验证机制,其注入了对`ProviderManager`的引用,并将调用它来处理其身份验证请求。
你需要的providers有时候可能会被认证机制(authentication mechanisms)调用,而其他时候,它们将取决于特定的身份验证机制(a specific authentication mechanism)。
例如，`DaoAuthenticationProvider`和`LdapAuthenticationProvider`与提交简单的用户名/密码身份验证请求的任何机制都兼容,因此,通常和基于表单的登录认证或HTTP Basic身份验证一起使用.
另一方面，某些身份验证机制会创建一个身份验证请求对象，该对象只能由一种类型的`AuthenticationProvider`。
一个示例就是`JA-SIG CAS`，它使用服务票证的概念，因此只能由进行身份验证`CasAuthenticationProvider`。

##### Erasing Credentials on Successful Authentication
默认情况下（从Spring Security 3.1开始），`ProviderManager`它将尝试从`Authentication`成功的身份验证请求返回的对象中清除所有敏感的凭据信息。这样可以防止将密码之类的信息保留的时间过长。

例如，在使用用户对象的缓存来提高无状态应用程序的性能时，这可能会导致问题。
如果`Authentication`包含对缓存中某个对象（例如`UserDetails`实例）的引用，并且已删除其凭据，则将无法再针对缓存的值进行身份验证.
如果使用缓存，则需要考虑到这一点。
一个明显的解决方案是先在高速缓存实现中,或在`AuthenticationProvider`创建返回`Authentication`的对象中,首先创建对象的副本。
或者，您可以在`ProviderManager`禁用该`eraseCredentialsAfterAuthentication`属性。

##### DaoAuthenticationProvider
`Spring Security`中最简单的`AuthenticationProvider`实现方法是`DaoAuthenticationProvider`.也是框架最早支持的方法之一。
它利用`UserDetailsService`（作为DAO）查找用户名，密码和`GrantedAuthority`
只需将比较`UsernamePasswordAuthenticationToken`提交的密码与`UserDetailsService`加载的密码，即可对用户进行身份验证

```xml
<beans>
<bean id="daoAuthenticationProvider"
    class="org.springframework.security.authentication.dao.DaoAuthenticationProvider">
<property name="userDetailsService" ref="inMemoryDaoImpl"/>
<property name="passwordEncoder" ref="passwordEncoder"/>
</bean>

<bean id="inMemoryDaoImpl"
 class="org.springframework.security.provisioning.InMemoryUserDetailsManager" >
....
</bean>
</beans>
```
`PasswordEncoder`是可选的。`PasswordEncoder`提供对`UserDetails`从配置的返回的对象中提供的密码进行编码和解码`UserDetailsService`。

#### UserDetailsService Implementations
大多数身份验证提供程序都利用`UserDetails`和`UserDetailsService`接口.`UserDetailsService`只是一个简单的方法:
```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
```
返回的`UserDetails`是一个接口，该接口提供了获取器，该获取器保证以非空方式提供身份验证信息，例如用户名，密码，已授予的权限以及用户帐户是启用还是禁用。
大多数身份验证提供程序(providers)会使用`UserDetailsService`,即使用户名和密码实际上没有用作身份验证决策的一部分.
他们使用`UserDetailsService`作为返回,可能仅仅只是为了获取`GrantedAuthority`信息,因为某些其他系统（例如LDAP或X.509或CAS等）承担了实际验证凭据的责任。

`UserDetailsService`可以很轻松的实现,因此用户应该使用自己选择的持久性策略轻松检索身份验证信息。`Spring Security`包含一些有用的基本实现.

##### In-Memory Authentication
创建自定义`UserDetailsService`实现,从持久化引擎中提取用户信息很简单,但是许多应用程序不需要这种复杂性.
如果你正在构建原型或刚刚开始集成`Spring Security`,而又不花时间配置数据库或者自定义一个`UserDetailsService`实现,则可采用下列配置
```xml
<user-service id="userDetailsService">
<!-- Password is prefixed with {noop} to indicate to DelegatingPasswordEncoder that
NoOpPasswordEncoder should be used. This is not safe for production, but makes reading
in samples easier. Normally passwords should be hashed using BCrypt -->
<user name="jimi" password="{noop}jimispassword" authorities="ROLE_USER, ROLE_ADMIN" />
<user name="bob" password="{noop}bobspassword" authorities="ROLE_USER" />
</user-service>
```
也支持使用外部属性文件：
```xml
<user-service id="userDetailsService" properties="users.properties"/>
```
属性文件应包含以下形式的条目
```properties
username=password,grantedAuthority[,grantedAuthority][,enabled|disabled]
```
例如
```properties
jimi=jimispassword,ROLE_USER,ROLE_ADMIN,enabled
bob=bobspassword,ROLE_USER,enabled
```

##### JdbcDaoImpl
`Spring Security`还包括一个`UserDetailsService`可从`JDBC`数据源获取身份验证信息的。
在内部使用`Spring JDBC`，因此它避免了仅用于存储用户详细信息的功能齐全的对象关系映射器（ORM）的复杂性。
如果您的应用程序确实使用了ORM工具，则您可能更愿意编写一个自定义`UserDetailsService`来重用您可能已经创建的映射文件。
返回`JdbcDaoImpl`，示例配置如下所示：
```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
<property name="driverClassName" value="org.hsqldb.jdbcDriver"/>
<property name="url" value="jdbc:hsqldb:hsql://localhost:9001"/>
<property name="username" value="sa"/>
<property name="password" value=""/>
</bean>

<bean id="userDetailsService"
    class="org.springframework.security.core.userdetails.jdbc.JdbcDaoImpl">
<property name="dataSource" ref="dataSource"/>
</bean>
```
您可以通过修改`DriverManagerDataSource`上面显示的内容使用不同的关系数据库管理系统。
您还可以使用从JNDI获得的全局数据源，就像其他任何Spring配置一样。

##### Authority Groups
默认情况下，`JdbcDaoImpl`假设权限直接映射到用户，则为单个用户加载权限。
另一种方法是将权限划分为组，然后将组分配给用户。有些人喜欢使用这种方法来管理用户权限。