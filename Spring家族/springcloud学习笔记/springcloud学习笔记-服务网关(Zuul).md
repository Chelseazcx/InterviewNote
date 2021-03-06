# springcloud学习笔记-服务网关(Zuul)

# 请求路由

构建一个简略的微服务架构，可能像下图这样：

[![img](https://ws2.sinaimg.cn/large/006tNc79ly1fqmei2skktj30ma0k2myr.jpg)](https://ws2.sinaimg.cn/large/006tNc79ly1fqmei2skktj30ma0k2myr.jpg)

我们使用 Spring Cloud Netflix 中的 Eureka 实现了服务注册中心以及服务注册与发现；而服务间通过 Ribbon 或 Feign 实现服务的消费以及均衡负载；通过 Spring Cloud Config 实现了应用多环境的外部化配置以及版本管理。为了使得服务集群更为健壮，使用 Hystrix 的融断机制来避免在微服务架构中个别服务出现异常时引起的故障蔓延。似乎一个微服务框架已经完成了。

我们还是少考虑了一个问题，外部的应用如何来访问内部各种各样的微服务呢？在微服务架构中，后端服务往往不直接开放给调用端，而是通过一个 API 网关根据请求的 URL，路由到相应的服务。当添加 API 网关后，在第三方调用端和服务提供方之间就创建了一面墙，这面墙直接与调用方通信进行权限控制，后将请求均衡分发给后台服务端。

## 为什么需要 API Gateway

**1、简化客户端调用复杂度**
在微服务架构模式下后端服务的实例数一般是动态的，对于客户端而言很难发现动态改变的服务实例的访问地址信息。因此在基于微服务的项目中为了简化前端的调用逻辑，通常会引入 API Gateway 作为轻量级网关，同时 API Gateway 中也会实现相关的认证逻辑从而简化内部服务之间相互调用的复杂度。

[![img](https://ws4.sinaimg.cn/large/006tNc79ly1fqmintl4smj30pi0e6mxw.jpg)](https://ws4.sinaimg.cn/large/006tNc79ly1fqmintl4smj30pi0e6mxw.jpg)

**2、数据裁剪以及聚合**
通常而言不同的客户端对于显示时对于数据的需求是不一致的，比如手机端或者 Web 端又或者在低延迟的网络环境或者高延迟的网络环境。

因此为了优化客户端的使用体验，API Gateway 可以对通用性的响应数据进行裁剪以适应不同客户端的使用需求。同时还可以将多个 API 调用逻辑进行聚合，从而减少客户端的请求数，优化客户端用户体验

**3、多渠道支持**
当然我们还可以针对不同的渠道和客户端提供不同的 API Gateway, 对于该模式的使用由另外一个大家熟知的方式叫 Backend for front-end, 在 Backend for front-end 模式当中，我们可以针对不同的客户端分别创建其 BFF，进一步了解 BFF 可以参考这篇文章：[Pattern: Backends For Frontends](http://samnewman.io/patterns/architectural/bff/)

[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqmdulzjxuj30ke0dc0tp.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqmdulzjxuj30ke0dc0tp.jpg)

**4、遗留系统的微服务化改造**
对于系统而言进行微服务改造通常是由于原有的系统存在或多或少的问题，比如技术债务，代码质量，可维护性，可扩展性等等。API Gateway 的模式同样适用于这一类遗留系统的改造，通过微服务化的改造逐步实现对原有系统中的问题的修复，从而提升对于原有业务响应力的提升。通过引入抽象层，逐步使用新的实现替换旧的实现。

[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqmduv84imj30v20hejta.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqmduv84imj30v20hejta.jpg)

在 Spring Cloud 体系中， Spring Cloud Zuul 就是提供负载均衡、反向代理、权限认证的一个 API Gateway。

## Spring Cloud Zuul

Spring Cloud Zuul 路由是微服务架构的不可或缺的一部分，提供动态路由、监控、弹性、安全等的边缘服务。Zuul 是 Netflix 出品的一个基于 JVM 路由和服务端的负载均衡器。

[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fr50yvfb4nj318g12ytdx.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fr50yvfb4nj318g12ytdx.jpg)

下面我们通过代码来了解 Zuul 是如何工作的

## 准备工作

在构建服务网关之前，我们先准备一下网关内部的微服务，可以直接使用前几篇编写的内容，比如：

- eureka
- producer
- consumer

在启动了 eureka、producer 和 consumer 的实例之后，所有的准备工作就以就绪，下面我们来试试使用 Spring Cloud Zuul 来实现服务网关的功能。

首先创建一个基础的 Spring Boot 项目，命名为：api-gateway。

### POM 配置

在`pom.xml`中引入以下依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

```

### 配置文件

在配置文件 application.yml 中加入服务名、端口号、Eureka 注册中心的地址：

```
spring:
  application:
    name: api-gateway
server:
  port: 14000
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7000/eureka/

```

### 启动类

使用`@EnableZuulProxy`注解开启 Zuul 的功能

```
@EnableZuulProxy
@SpringBootApplication
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}

```

到这里，一个基于 Spring Cloud Zuul 服务网关就已经构建完毕。启动该应用，一个默认的服务网关就构建完毕了，同时能在 Eureka 里看到这个服务。
[![img](https://ws1.sinaimg.cn/large/006tNc79ly1fqmf8vyiasj31kw0v90zp.jpg)](https://ws1.sinaimg.cn/large/006tNc79ly1fqmf8vyiasj31kw0v90zp.jpg)

### 测试

由于 Spring Cloud Zuul 在整合了 Eureka 之后，具备默认的服务路由功能，即：当我们这里构建的`api-gateway`应用启动并注册到 Eureka 之后，服务网关会发现上面我们启动的两个服务`producer`和`consumer`，这时候 Zuul 就会创建两个路由规则。每个路由规则都包含两部分，一部分是外部请求的匹配规则，另一部分是路由的服务 ID。针对当前示例的情况，Zuul 会创建下面的两个路由规则：

- 转发到`producer`服务的请求规则为：`/producer/**`
- 转发到`consumer`服务的请求规则为：`/consumer/**`

最后，我们可以通过访问`14000`端口的服务网关来验证上述路由的正确性：

- 比如访问：[http://localhost:14000/consumer/hello/windmt](http://localhost:14000/consumer/hello/windmt)，该请求将最终被路由到`consumer`的`/hello`接口上。

# 过滤器

本文我们将关注 Spring Cloud Zuul 的另一核心功能：过滤器（Filter）。

zuul中定义的四中不同生命的过滤器类型：

　　1. pre：路由之前
　　2. routing:路由之时
　　3. post:路由之后
　　4. error：发送错误调用


## Filter 的作用

每个客户端用户请求微服务应用提供的接口时，它们的访问权限往往都需要有一定的限制，系统并不会将所有的微服务接口都对它们开放。然而，目前的服务路由并没有限制权限这样的功能，所有请求都会被毫无保留地转发到具体的应用并返回结果。

为了实现对客户端请求的安全校验和权限控制，最简单和粗暴的方法就是为每个微服务应用都实现一套用于校验签名和鉴别权限的过滤器或拦截器。

不过，这样的做法并不可取，它会增加日后的系统维护难度，因为同一个系统中的各种校验逻辑很多情况下都是大致相同或类似的，这样的实现方式会使得相似的校验逻辑代码被分散到了各个微服务中去，冗余代码的出现是我们不希望看到的。所以，比较好的做法是将这些校验逻辑剥离出去，构建出一个独立的鉴权服务。

在完成了剥离之后，有不少开发者会直接在微服务应用中通过调用鉴权服务来实现校验，但是这样的做法仅仅只是解决了鉴权逻辑的分离，并没有在本质上将这部分不属于业余的逻辑拆分出原有的微服务应用，冗余的拦截器或过滤器依然会存在。

对于这样的问题，更好的做法是通过前置的网关服务来完成这些非业务性质的校验。由于网关服务的加入，外部客户端访问我们的系统已经有了统一入口，既然这些校验与具体业务无关，那何不在请求到达的时候就完成校验和过滤，而不是转发后再过滤而导致更长的请求延迟。同时，通过在网关中完成校验和过滤，微服务应用端就可以去除各种复杂的过滤器和拦截器了，这使得微服务应用的接口开发和测试复杂度也得到了相应的降低。

为了在 API 网关中实现对客户端请求的校验，我们将需要使用到 Spring Cloud Zuul 的另外一个核心功能：**过滤器**。

Zuul 允许开发者在 API 网关上通过定义过滤器来实现对请求的拦截与过滤，实现的方法非常简单。

## Filter 的生命周期

Filter 的生命周期有 4 个，分别是 “PRE”、“ROUTING”、“POST” 和“ERROR”，整个生命周期可以用下图来表示
[![img](https://ws2.sinaimg.cn/large/006tNc79ly1fqmg1wtyhdj30pl0fqdgt.jpg)](https://ws2.sinaimg.cn/large/006tNc79ly1fqmg1wtyhdj30pl0fqdgt.jpg)

Zuul 大部分功能都是通过过滤器来实现的，这些过滤器类型对应于请求的典型生命周期。

- **PRE：** 这种过滤器在请求被路由之前调用。我们可利用这种过滤器实现身份验证、在集群中选择请求的微服务、记录调试信息等。
- **ROUTING：** 这种过滤器将请求路由到微服务。这种过滤器用于构建发送给微服务的请求，并使用 Apache HttpClient 或 Netfilx Ribbon 请求微服务。
- **POST：** 这种过滤器在路由到微服务以后执行。这种过滤器可用来为响应添加标准的 HTTP Header、收集统计信息和指标、将响应从微服务发送给客户端等。
- **ERROR：** 在其他阶段发生错误时执行该过滤器。 除了默认的过滤器类型，Zuul 还允许我们创建自定义的过滤器类型。例如，我们可以定制一种 STATIC 类型的过滤器，直接在 Zuul 中生成响应，而不将请求转发到后端的微服务。

## Zuul 中默认实现的 Filter

| 类型    | 顺序   | 过滤器                     | 功能                       |
| ----- | ---- | ----------------------- | ------------------------ |
| pre   | -3   | ServletDetectionFilter  | 标记处理 Servlet 的类型         |
| pre   | -2   | Servlet30WrapperFilter  | 包装 HttpServletRequest 请求 |
| pre   | -1   | FormBodyWrapperFilter   | 包装请求体                    |
| route | 1    | DebugFilter             | 标记调试标志                   |
| route | 5    | PreDecorationFilter     | 处理请求上下文供后续使用             |
| route | 10   | RibbonRoutingFilter     | serviceId 请求转发           |
| route | 100  | SimpleHostRoutingFilter | url 请求转发                 |
| route | 500  | SendForwardFilter       | forward 请求转发             |
| post  | 0    | SendErrorFilter         | 处理有错误的请求响应               |
| post  | 1000 | SendResponseFilter      | 处理正常的请求响应                |

## 禁用指定的 Filter

可以在 application.yml 中配置需要禁用的 filter，格式为`zuul...disable=true`。
比如要禁用`org.springframework.cloud.netflix.zuul.filters.post.SendResponseFilter`就设置

```
zuul:
  SendResponseFilter:
    post:
      disable: true

```

## 自定义 Filter

我们假设有这样一个场景，因为服务网关应对的是外部的所有请求，为了避免产生安全隐患，我们需要对请求做一定的限制，比如请求中含有 Token 便让请求继续往下走，如果请求不带 Token 就直接返回并给出提示。

首先自定义一个 Filter，继承 ZuulFilter 抽象类，在 run() 方法中验证参数是否含有 Token，具体如下：

```
public class TokenFilter extends ZuulFilter {

    /**
     * 过滤器的类型，它决定过滤器在请求的哪个生命周期中执行。
     * 这里定义为pre，代表会在请求被路由之前执行。
     *
     * @return
     */
    @Override
    public String filterType() {
        return "pre";
    }

    /**
     * filter执行顺序，通过数字指定。
     * 数字越大，优先级越低。
     *
     * @return
     */
    @Override
    public int filterOrder() {
        return 0;
    }

    /**
     * 判断该过滤器是否需要被执行。这里我们直接返回了true，因此该过滤器对所有请求都会生效。
     * 实际运用中我们可以利用该函数来指定过滤器的有效范围。
     *
     * @return
     */
    @Override
    public boolean shouldFilter() {
        return true;
    }

    /**
     * 过滤器的具体逻辑
     *
     * @return
     */
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();

        String token = request.getParameter("token");
        if (token == null || token.isEmpty()) {
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            ctx.setResponseBody("token is empty");
        }
        return null;
    }
}

```

在上面实现的过滤器代码中，我们通过继承`ZuulFilter`抽象类并重写了下面的四个方法来实现自定义的过滤器。这四个方法分别定义了：

- `filterType()`：过滤器的类型，它决定过滤器在请求的哪个生命周期中执行。这里定义为`pre`，代表会在请求被路由之前执行。
- `filterOrder()`：过滤器的执行顺序。当请求在一个阶段中存在多个过滤器时，需要根据该方法返回的值来依次执行。通过数字指定，数字越大，优先级越低。
- `shouldFilter()`：判断该过滤器是否需要被执行。这里我们直接返回了`true`，因此该过滤器对所有请求都会生效。实际运用中我们可以利用该函数来指定过滤器的有效范围。
- `run()`：过滤器的具体逻辑。这里我们通过`ctx.setSendZuulResponse(false)`令 Zuul 过滤该请求，不对其进行路由，然后通过`ctx.setResponseStatusCode(401)`设置了其返回的错误码，当然我们也可以进一步优化我们的返回，比如，通过`ctx.setResponseBody(body)`对返回 body 内容进行编辑等。

在实现了自定义过滤器之后，它并不会直接生效，我们还需要为其创建具体的 Bean 才能启动该过滤器，比如，在应用主类中增加如下内容：

```
@EnableZuulProxy
@SpringBootApplication
public class ApiGatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }

    @Bean
    public TokenFilter tokenFilter() {
        return new TokenFilter();
    }
}

```

在对`api-gateway`服务完成了上面的改造之后，我们可以重新启动它，并发起下面的请求，对上面定义的过滤器做一个验证：

- 访问 [http://localhost:14000/consumer/hello/windmt](http://localhost:14000/consumer/hello/windmt) 返回 401 错误和`token is empty`
- 访问 [http://localhost:14000/consumer/hello/windmt?token=token](http://localhost:14000/consumer/hello/windmt?token=token) 正确路由到`consumer`的`/hello`接口，并返回`Hello, windmt`

我们可以根据自己的需要在服务网关上定义一些与业务无关的通用逻辑实现对请求的过滤和拦截，比如：签名校验、权限校验、请求限流等功能。

# 原理详解

## 服务路由配置

只需要通过 path 与 serviceId 参数对的方式进行配置即可。我们可以将API网关看做服务治理下的一个普通微服务应用。在Eureka的帮助下，网关本身维护系统中所有的 serviceId 与实际映射的关系。

## 路由的默认规则

采用服务名作为外部请求的前缀。有一些服务我们不需要对外开发也被外部访问到了。这个时候我们可以使用 zuul.ignore-services 参数来设置一个服务名匹配表达式来定义不自动创建路由的规则。

## 自定义路由映射关系

具有版本号前缀的url路径，我们就可以很同意的通过路径表达式来归类和管理具有版本信息的微服务了。

## 路径匹配，路由前缀，cookie与头信息

不论是使用传统配置方式还是服务路由的配置方式，我们都需要为每个路由定义匹配表达式。为路由规则增加前缀信息。

# 过滤器源码

在Spring Cloud Zuul中实现的过滤器必须包含4个基本特征：过滤类型、执行顺序、执行条件、具体操作。

```
String filterType();    
int filterOrder();
boolean shouldFilter();
Object run();
```

各自的含义与功能总结如下：

- filterType：该函数需要返回一个字符串来代表过滤器的类型，而这个类型就是在HTTP请求过程中定义的各个阶段。在Zuul中默认定义了四种不同生命周期的过滤器类型，具体如下：
  - pre：可以在请求被路由之前调用。
  - routing：在路由请求时候被调用。
  - post：在routing和error过滤器之后被调用。
  - error：处理请求时发生错误时被调用。
- filterOrder：通过int值来定义过滤器的执行顺序，数值越小优先级越高。
- shouldFilter：返回一个boolean类型来判断该过滤器是否要执行。我们可以通过此方法来指定过滤器的有效范围。
- run：过滤器的具体逻辑。在该函数中，我们可以实现自定义的过滤逻辑，来确定是否要拦截当前的请求，不对其进行后续的路由，或是在请求路由返回结果之后，对处理结果做一些加工等。

## 过滤器

#### pre过滤器

- ServletDetectionFilter：它的执行顺序为-3，是最先被执行的过滤器。该过滤器总是会被执行，主要用来检测当前请求是通过Spring的DispatcherServlet处理运行，还是通过ZuulServlet来处理运行的。它的检测结果会以布尔类型保存在当前请求上下文的isDispatcherServletRequest参数中，这样在后续的过滤器中，我们就可以通过RequestUtils.isDispatcherServletRequest()和RequestUtils.isZuulServletRequest()方法判断它以实现做不同的处理。一般情况下，发送到API网关的外部请求都会被Spring的DispatcherServlet处理，除了通过/zuul/*路径访问的请求会绕过DispatcherServlet，被ZuulServlet处理，主要用来应对处理大文件上传的情况。另外，对于ZuulServlet的访问路径/zuul/*，我们可以通过zuul.servletPath参数来进行修改。
- Servlet30WrapperFilter：它的执行顺序为-2，是第二个执行的过滤器。目前的实现会对所有请求生效，主要为了将原始的HttpServletRequest包装成Servlet30RequestWrapper对象。
- FormBodyWrapperFilter：它的执行顺序为-1，是第三个执行的过滤器。该过滤器仅对两种类请求生效，第一类是Content-Type为application/x-www-form-urlencoded的请求，第二类是Content-Type为multipart/form-data并且是由Spring的DispatcherServlet处理的请求（用到了ServletDetectionFilter的处理结果）。而该过滤器的主要目的是将符合要求的请求体包装成FormBodyRequestWrapper对象。
- DebugFilter：它的执行顺序为1，是第四个执行的过滤器。该过滤器会根据配置参数zuul.debug.request和请求中的debug参数来决定是否执行过滤器中的操作。而它的具体操作内容则是将当前的请求上下文中的debugRouting和debugRequest参数设置为true。由于在同一个请求的不同生命周期中，都可以访问到这两个值，所以我们在后续的各个过滤器中可以利用这两值来定义一些debug信息，这样当线上环境出现问题的时候，可以通过请求参数的方式来激活这些debug信息以帮助分析问题。另外，对于请求参数中的debug参数，我们也可以通过zuul.debug.parameter来进行自定义。
- PreDecorationFilter：它的执行顺序为5，是pre阶段最后被执行的过滤器。该过滤器会判断当前请求上下文中是否存在forward.to和serviceId参数，如果都不存在，那么它就会执行具体过滤器的操作（如果有一个存在的话，说明当前请求已经被处理过了，因为这两个信息就是根据当前请求的路由信息加载进来的）。而它的具体操作内容就是为当前请求做一些预处理，比如：进行路由规则的匹配、在请求上下文中设置该请求的基本信息以及将路由匹配结果等一些设置信息等，这些信息将是后续过滤器进行处理的重要依据，我们可以通过RequestContext.getCurrentContext()来访问这些信息。另外，我们还可以在该实现中找到一些对HTTP头请求进行处理的逻辑，其中包含了一些耳熟能详的头域，比如：X-Forwarded-Host、X-Forwarded-Port。另外，对于这些头域的记录是通过zuul.addProxyHeaders参数进行控制的，而这个参数默认值为true，所以Zuul在请求跳转时默认地会为请求增加X-Forwarded-*头域，包括：X-Forwarded-Host、X-Forwarded-Port、X-Forwarded-For、X-Forwarded-Prefix、X-Forwarded-Proto。我们也可以通过设置zuul.addProxyHeaders=false关闭对这些头域的添加动作。

#### route过滤器

- RibbonRoutingFilter：它的执行顺序为10，是route阶段第一个执行的过滤器。该过滤器只对请求上下文中存在serviceId参数的请求进行处理，即只对通过serviceId配置路由规则的请求生效。而该过滤器的执行逻辑就是面向服务路由的核心，它通过使用Ribbon和Hystrix来向服务实例发起请求，并将服务实例的请求结果返回。
- SimpleHostRoutingFilter：它的执行顺序为100，是route阶段第二个执行的过滤器。该过滤器只对请求上下文中存在routeHost参数的请求进行处理，即只对通过url配置路由规则的请求生效。而该过滤器的执行逻辑就是直接向routeHost参数的物理地址发起请求，从源码中我们可以知道该请求是直接通过httpclient包实现的，而没有使用Hystrix命令进行包装，所以这类请求并没有线程隔离和断路器的保护。
- SendForwardFilter：它的执行顺序为500，是route阶段第三个执行的过滤器。该过滤器只对请求上下文中存在forward.to参数的请求进行处理，即用来处理路由规则中的forward本地跳转配置。

#### post过滤器

- SendErrorFilter：它的执行顺序为0，是post阶段第一个执行的过滤器。该过滤器仅在请求上下文中包含error.status_code参数（由之前执行的过滤器设置的错误编码）并且还没有被该过滤器处理过的时候执行。而该过滤器的具体逻辑就是利用请求上下文中的错误信息来组织成一个forward到API网关/error错误端点的请求来产生错误响应。
- SendResponseFilter：它的执行顺序为1000，是post阶段最后执行的过滤器。该过滤器会检查请求上下文中是否包含请求响应相关的头信息、响应数据流或是响应体，只有在包含它们其中一个的时候就会执行处理逻辑。而该过滤器的处理逻辑就是利用请求上下文的响应信息来组织需要发送回客户端的响应内容。







