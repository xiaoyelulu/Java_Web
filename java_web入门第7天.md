## 基础登录功能
### 登录功能
![Alt text](pictures/java_web入门第7天.png)

### 登录校验(重点)
![Alt text](pictures/java_web入门第7天01.png)
![Alt text](pictures/java_web入门第7天02.png)
### 会话技术
- 会话：用户打开浏览器，访问 web 服务器的资源，会话建立，直到有一方断开连接，会话结束。在一次会话中可以包含多次请求和响应。
- 会话跟踪：一种维护浏览器状态的方法，服务器需要识别多次请求是否来自于同一浏览器，以便在同一次会话的多次请求间共享数据。
- 会话跟踪方案：
    - 客户端会话跟踪技术：Cookie
    - 服务端会话跟踪技术：Session
    - 令牌技术
![Alt text](pictures/java_web入门第7天03.png)
- 跨域区分三个维度：协议、IP/域名、端口
### JWT 令牌
#### **简介：**
- 全称：JSON  Web  Token （https://jwt.io/）
- 定义了一种简洁的、自包含的格式，用于在通信双方以json数据格式安全的传输信息。由于数字签名的存在，这些信息是可靠的。
- 组成：
    - 第一部分：Header（头）， 记录令牌类型、签名算法等。 例如：`{"alg":"HS256","type":"JWT"}`
    - 第二部分：Payload（有效载荷），携带一些自定义信息、默认信息等。 例如：`{"id":"1","username":"Tom"}`
    - 第三部分：Signature（签名），防止 Token 被篡改、确保安全性。将 header、payload，并加入指定秘钥，通过指定签名算法计算而来。
- Base64： 是一种基于 64 个可打印字符 `（A-Z  a-z  0-9  +  /）` 来表示二进制数据的编码方式。

#### **场景：登录认证**
1. 登录成功后，生成令牌
2. 后续每个请求，都要携带JWT令牌，系统在每次请求处理之前，先校验令牌，通过后，再处理
![Alt text](pictures/java_web入门第7天04.png)

#### **JWT - 生成：**

![Alt text](pictures/java_web入门第7天05.png)

#### **JWT - 校验：**
```java
@Test
public void parseJwt(){
    Claims claims = Jwts.parser()
        .setSigningKey(“itheima”) //指定签名秘钥
        .parseClaimsJws(“eyJhbGciOiJIUzI1NiJ9.eyJpZCI6MSwiZXhwIjoxNjU5OTk1NTE3LCJ1c2VybmFtZSI6IlRvbSJ9.EUTfeqPkGslekdKBezcWCe7a7xbcIIwB1MXlIccTMwo”) //解析令牌
        .getBody();
    System.out.println(claims);
    }
```
注意事项：
1. JWT 校验时使用的签名秘钥，必须和生成 JWT 令牌时使用的秘钥是配套的。
2. 如果 JWT 令牌解析校验时报错，则说明 JWT 令牌被篡改 或 失效了，令牌非法。 

![Alt text](pictures/java_web入门第7天06.png)

#### **登录-生成令牌**
- 引入JWT令牌操作工具类。
- 登录完成后，调用工具类生成JWT令牌，并返回。

### 过滤器 Filter
**概述：**
1. 概念：Filter 过滤器，是 JavaWeb 三大组件( Servlet、Filter、Listener )之一。
2. 过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能。
3. 过滤器一般完成一些通用的操作，比如：登录校验、统一编码处理、敏感字符处理等。
![Alt text](pictures/java_web入门第7天07.png)

#### 快速入门
1. 定义 Filter：定义一个类，实现 Filter 接口，并重写其所有方法。
2. 配置 Filter：Filter 类上加 `@WebFilter(urlPatterns="/*")` 注解，配置拦截资源的路径。引导类上加 `@ServletComponentScan` 开启 Servlet 组件支持。
![Alt text](pictures/java_web入门第7天08.png)
#### 详解(执行流程、拦截路径、过滤器链)
**执行流程：**
![Alt text](pictures/java_web入门第7天09.png)

**Filter 拦截路径**
![Alt text](pictures/java_web入门第7天10.png)

**过滤器链**
- 介绍：一个 web 应用中，可以配置多个过滤器，这多个过滤器就形成了一个过滤器链。
- 顺序： 注解配置的 Filter ，优先级是按照过滤器类名（字符串）的自然排序。
![Alt text](pictures/java_web入门第7天11.png)
#### 登录校验 - Filter
- 所有的请求，拦截到了之后，都需要校验令牌吗？ 有一个例外，登录请求
- 拦截到请求后，什么情况下才可以放行，执行业务操作？ 有令牌，且令牌校验通过（合法）；否则都返回未登录错误结果

**登录校验Filter-流程**
![Alt text](pictures/java_web入门第7天12.png)
### 拦截器 Interceptor
#### 简介 & 快速入门
**概述：**
- 概念：是一种动态拦截方法调用的机制，类似于过滤器。Spring 框架中提供的，用来动态拦截控制器方法的执行。
- 作用：拦截请求，在指定的方法调用前后，根据业务需要执行预先设定的代码。
![Alt text](pictures/java_web入门第7天13.png)

**Interceptor 快速入门**
- 定义拦截器，实现 HandlerInterceptor 接口，并重写其所有方法。
- 注册拦截器
![Alt text](pictures/java_web入门第7天14.png)

ctr + o 重写函数
#### 详解(拦截路径、执行流程)
- 拦截器可以根据需求，配置不同的拦截路径：
![Alt text](pictures/java_web入门第7天15.png)
![Alt text](pictures/java_web入门第7天16.png)
- `Filter` 与 `Interceptor` 区别
    - 接口规范不同：过滤器需要实现 `Filter` 接口，而拦截器需要实现 `HandlerInterceptor` 接口。
    - 拦截范围不同：过滤器 `Filter` 会拦截所有的资源，而 ` Interceptor`只会拦截 Spring 环境中的资源。
  
#### 登录校验- Interceptor
- 放行，return true；不放行 return false
### 异常处理
1. 出现异常，该如何处理?
- 方案一： 在 Controller 的方法中进行 try …catch 处理
- 方案二： 全局异常处理器 (简单、优雅，推荐)
2. 全局异常处理器 `@RestControllerAdvice = @ControllerAdvice + @ResponseBody`
![Alt text](pictures/java_web入门第7天17.png)
```java
@RestControllerAdvice
public class GlobalExceptionHandle {
    @ExceptionHandler(Exception.class) //捕获所有异常
    public Result ex(Exception ex){
        ex.printStackTrace();
        return Result.error("对不起，操作失败，请联系管理员");
    }
}
```
3. 全局异常处理器： 
    - ` @RestControllerAdvice`
    - ` @ExceptionHandler`

# 第 7 章 事务管理 & AOP
## 事务管理
### Spring 事务管理
- 注解：`@Transactional`
- 位置：业务（service）层的方法上、类上、接口上
- 作用：将当前方法交给 spring 进行事务管理，方法执行前，开启事务；成功执行完毕，提交事务；出现异常，回滚事务

![Alt text](pictures/java_web入门第7天18.png)
```yaml
#开启事务管理日志
logging:
  level:
    org.springframework.jdbc.support.JdbcTransactionManager: debug
```
### 事务进阶
#### rollbackFor
默认情况下，只有出现 `RuntimeException` 才回滚异常。 `rollbackFor` 属性用于控制出现何种异常类型，回滚事务。
- `@Transactional(rollbackFor = Exception.class)`

#### propagation
事务传播行为：指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何进行事务控制。
- `@Transactional(propagation = Propagation.REQUIRED)`，默认 REQUIRED 参数
![Alt text](pictures/java_web入门第7天19.png)
- 事务属性 - 传播行为
    - REQUIRED ：大部分情况下都是用该传播行为即可。
    - REQUIRES_NEW ：当我们不希望事务之间相互影响时，可以使用该传播行为。比如：下订单前需要记录日志，不论订单保存成功与否，都需要保证日志记录能够记录成功。

## AOP 基础
### AOP概述
- AOP：Aspect Oriented Programming（面向切面编程、面向方面编程），其实就是面向特定方法编程。
- 实现：
    - 动态代理是面向切面编程最主流的实现。而 SpringAOP 是 Spring 框架的高级技术，旨在管理 bean 对象的过程中，主要通过底层的动态代理机制，对特定的方法进行编程。
- 场景：
    - 记录操作日志
    - 权限控制
    - 事务管理
- 优势：
    - 代码无侵入
    - 减少重复代码
    - 提高开发效率
    - 维护方便
### AOP核心概念
- 连接点：JoinPoint，可以被AOP控制的方法（暗含方法执行时的相关信息）
- 通知：Advice，指哪些重复的逻辑，也就是共性功能（最终体现为一个方法）
- 切入点：PointCut，匹配连接点的条件，通知仅会在切入点方法执行时被应用
- 切面：Aspect，描述通知与切入点的对应关系（通知+切入点）
-  目标对象：Target，通知所应用的对象
## AOP 进阶
### 通知类型
#### 通知类型：
1. @Around：环绕通知，此注解标注的通知方法在目标方法前、后都被执行
2. @Before：前置通知，此注解标注的通知方法在目标方法前被执行
3. @After ：后置通知，此注解标注的通知方法在目标方法后被执行，无论是否有异常都会执行
4. @AfterReturning ： 返回后通知，此注解标注的通知方法在目标方法后被执行，有异常不会执行
5. @AfterThrowing ： 异常后通知，此注解标注的通知方法发生异常后执行

注意事项：
- @Around 环绕通知需要自己调用 ProceedingJoinPoint.proceed() 来让原始方法执行，其他通知不需要考虑目标方法执行
- @Around 环绕通知方法的返回值，必须指定为 Object，来接收原始方法的返回值。
#### @PointCut
该注解的作用是将公共的切点表达式抽取出来，需要用到时引用该切点表达式即可。
- private：仅能在当前切面类中引用该表达式
- public：在其他外部的切面类中也可以引用该表达式
### 通知顺序
#### 执行顺序
1. 不同切面类中，默认按照切面类的类名字母排序：
   - 目标方法前的通知方法：字母排名靠前的先执行
   - 目标方法后的通知方法：字母排名靠前的后执行
2. 用 @Order(数字) 加在切面类上来控制顺序
   - 目标方法前的通知方法：数字小的先执行
   - 目标方法后的通知方法：数字小的后执行

### 切入点表达式
- 切入点表达式：描述切入点方法的一种表达式
- 作用：主要用来决定项目中的哪些方法需要加入通知
- 常见形式：
  - execution(……)：根据方法的签名来匹配
  - @annotation(……) ：根据注解匹配

#### 切入点表达式-execution
execution 主要根据方法的返回值、包名、类名、方法名、方法参数等信息来匹配，语法为：

`execution(访问修饰符?  返回值  包名.类名.?方法名(方法参数) throws 异常?)
`

其中带 ? 的表示可以省略的部分
- 访问修饰符：可省略（比如: public、protected）
- 包名.类名： 可省略
- throws 异常：可省略（注意是方法上声明抛出的异常，不是实际抛出的异常）

可以使用通配符描述切入点
- \* ：单个独立的任意符号，可以通配任意返回值、包名、类名、方法名、任意类型的一个参数，也可以通配包、类、方法名的一部分
- .. ：多个连续的任意符号，可以通配任意层级的包，或任意类型、任意个数的参数

注意事项：根据业务需要，可以使用 且（&&）、或（||）、非（!） 来组合比较复杂的切入点表达式。

#### 切入点表达式-@annotation
@annotation 切入点表达式，用于匹配标识有特定注解的方法。

`@annotation(com.itheima.anno.Log) @annotation(注解全类名)
` 
### 连接点
在 Spring 中用 JoinPoint 抽象了连接点，用它可以获得方法执行时的相关信息，如目标类名、方法名、方法参数等。
- 对于 @Around 通知，获取连接点信息只能使用  ProceedingJoinPoint
- 对于其他四种通知，获取连接点信息只能使用 JoinPoint ，它是 ProceedingJoinPoint 的父类型


# 第 8 章 Spring Boot 原理篇
## 配置优先级
越往下优先级越高：
1. application.yaml（忽略）
2. application.yml
3. application.properties
4. java系统属性（-Dxxx=xxx）
5. 命令行参数（--xxx=xxx）
## Bean管理
### 获取bean
![Alt text](pictures/java_web入门第7天20.png)

注意事项：上述所说的 【Spring项目启动时，会把其中的bean都创建好】还会受到作用域及延迟初始化影响，这里主要针对于 默认的单例非延迟加载的bean而言。

### bean作用域
|作用域 | 说明|
|:--------:| :---------:|
|singleton |容器内同 名称 的 bean 只有一个实例（单例）（默认）|
|prototype|每次使用该 bean 时会创建新的实例（非单例）|

可以通过 @Scope 注解来进行配置作用域：
```java
@Scope("prototype")
@RestController
@RequestMapping("/depts")
public class DeptController {
}
```
注意事项：
- 默认 singleton 的 bean ，在容器启动时被创建，可以使用 @Lazy 注解来延迟初始化（延迟到第一次使用时）。
- prototype 的 bean ，每一次使用该 bean 的时候都会创建一个新的实例。
- 实际开发当中，绝大部分的 Bean 是单例的，也就是说绝大部分 Bean 不需要配置 scope 属性。
### 第三方bean
![Alt text](pictures/java_web入门第7天21.png)

@Component 及衍生注解 与 @Bean注解使用场景？
- 项目中自定义的，使用 @Component 及其衍生注解
- 项目中引入第三方的，使用 @Bean 注解
## SpringBoot原理
### 起步依赖
自动传递依赖
### 自动配置
SpringBoot 的自动配置就是当 spring 容器启动后，一些配置类、bean 对象就自动存入到了 IOC 容器中，不需要我们手动去声明，从而简化了开发，省去了繁琐的配置操作。

1. 方案一：@ComponentScan 组件扫描。繁琐
2. 方案二：@Import 导入。使用@Import导入的类会被Spring加载到IOC容器中，导入形式主要有以下几种：
    - 导入 普通类
    - 导入 配置类
    - 导入 ImportSelector 接口实现类
    - @EnableXxxx注解，封装@Import注解 

![Alt text](pictures/java_web入门第7天22.png)
![Alt text](pictures/java_web入门第7天23.png)

#### @Conditional
 - 作用：按照一定的条件进行判断，在满足给定条件后才会注册对应的bean对象到Spring IOC容器中。
 - 位置：方法、类
 - `@Conditional` 本身是一个父注解，派生出大量的子注解：
    - `@ConditionalOnClass`：判断环境中是否有对应字节码文件，才注册 bean 到 IOC 容器。
    - `@ConditionalOnMissingBean`：判断环境中没有对应的 bean（类型 或 名称） ，才注册bean 到 IOC 容器。
    - `@ConditionalOnProperty`：判断配置文件中有对应属性和值，才注册 bean 到 IOC 容器。
```java
// SpringBoot会根据@Conditional注解条件装配
@Bean
@ConditionalOnClass(name = "io.jsonwebtoken.Jwts") //当前环境存在指定的这个类时，才声明该bean 
public HeaderParser headerParser(){...}

@Bean
@ConditionalOnMissingBean //当不存在当前类型的bean时，才声明该bean 
public HeaderParser headerParser(){...}

@Bean
@ConditionalOnProperty(name = "name",havingValue = "itheima") //配置文件中存在对应的属性和值，才注册bean到IOC容器。
public HeaderParser headerParser(){...}
```
### 自定义 starter
在实际开发中，经常会定义一些公共组件，提供给各个项目团队使用。而在SpringBoot的项目中，一般会将这些公共组件封装为SpringBoot 的 starter。
![Alt text](pictures/java_web入门第7天24.png)  
![Alt text](pictures/java_web入门第7天25.png)
![Alt text](pictures/java_web入门第7天26.png)