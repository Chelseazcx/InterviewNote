# Spring

[TOC]

## 1. Spring IOC、AOP的理解、实现的原理，以及优点

Spring的IoC容器是Spring的核心，Spring AOP是spring框架的重要组成部分

### IOC

- **我的理解**
  - 正常的情况下，比如有一个类，在类里面有方法（不是静态的方法），调用类里面的方法，创建类的对象，使用对象调用方法，创建类对象的过程，需要new出来对象
  - 通过控制反转，把对象的创建不是通过new方式实现，而是交给Spring配置创建类对象
  - IOC的意思是控件反转也就是由容器控制程序之间的关系，这也是spring的优点所在，把控件权交给了外部容器，之前的写法，由程序代码直接操控，而现在控制权由应用代码中转到了外部容器，控制权的转移是所谓反转。换句话说之前用new的方式获取对象，现在由spring给你至于怎么给你就是di了。
- **Spring IOC实现原理**
  - 创建xml配置文件，配置要创建的对象类
  - 通过反射创建实例；
  - 获取需要注入的接口实现类并将其赋值给该接口。
- **优点**
  - 解耦合，开发更方便组织分工
  - 高层不依赖于底层（依赖倒置）
  - 使应用更容易测试
  - 因为把对象生成放在了XML里定义，所以当我们需要换一个实现子类将会变成很简单（一般这样的对象都是现实于某种接口的），只要修改XML就可以了，这样我们甚至可以实现对象的热插拨

### AOP

- **我的理解**
  - AOP（Aspect Oriented Programming ）称为面向切面编程，扩展功能不是修改源代码实现，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待，Struts2的拦截器设计就是基于AOP的思想，是个比较经典的例子。
  - 面向切面编程（aop）是对面向对象编程（oop）的补充
  - 面向切面编程提供声明式事务管理
  - AOP就是典型的代理模式的体现
- **Spring AOP实现原理**
  - 动态代理（利用**反射和动态编译**将代理模式变成动态的）
  - JDK的动态代理
    - JDK内置的Proxy动态代理可以在运行时动态生成字节码，而没必要针对每个类编写代理类
    - JDKProxy返回动态代理类，是目标类所实现接口的另一个实现版本，它实现了对目标类的代理（如同UserDAOProxy与UserDAOImp的关系）
  - cglib动态代理
    - CGLibProxy返回的动态代理类，则是目标代理类的一个子类（代理类扩展了UserDaoImpl类）
    - cglib继承被代理的类，重写方法，织入通知，动态生成字节码并运行
  - JDK动态代理与cglib实现的区别
    - JDK动态代理只能对实现了接口的类生成代理，而不能针对类.
    - cglib是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法因为是继承，所以该类或方法最好不要声明成final。
    - JDK代理是不需要以来第三方的库，只要JDK环境就可以进行代理
    - cglib必须依赖于cglib的类库，但是它需要类来实现任何接口代理的是指定的类生成一个子类，覆盖其中的方法，是一种继承
    - CGLib创建的动态代理对象性能比JDK创建的动态代理对象的性能高不少，但是CGLib在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用CGLib合适
-  **优点**
  - 各个步骤之间的良好隔离性
  - 源代码无关性
  - 松耦合
  - 易扩展
  - 代码复用

## 2. 什么是依赖注入，注入的方式有哪些

- DI（依赖注入）
  - 所谓依赖注入，就是把底层类作为参数传入上层类，实现上层类对下层类的控制。DI依赖注入，向类里面属性注入值 ，依赖注入不能单独存在，需要在IOC基础上完成操作。
- - 使用set方法注入
  - 使用有参构造注入
  - 使用接口注入
  - 注解注入(@Autowire)

## 3. Spring IOC初始化过程

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/bean-init2.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/bean-init2.png)

IOC容器的初始化分为三个过程实现：

- 第一个过程是Resource资源定位。这个Resouce指的是BeanDefinition的资源定位。这个过程就是容器找数据的过程，就像水桶装水需要先找到水一样。
- 第二个过程是BeanDefinition的载入过程。这个载入过程是把用户定义好的Bean表示成Ioc容器内部的数据结构，而这个容器内部的数据结构就是BeanDefition。
- 第三个过程是向IOC容器注册这些BeanDefinition的过程，这个过程就是将前面的BeanDefition保存到HashMap中的过程。

## 4. 项目中Spring AOP用在什么地方，为什么这么用，切点，织入，通知用自己的话描述一下

- **Joinpoint（连接点）** 类里面可以被增强的方法，这些方法称为连接点
- **Pointcut（切入点）** 所谓切入点是指我们要对哪些Joinpoint进行拦截的定义
- **Advice（通知/增强）** 所谓通知是指拦截到Joinpoint之后所要做的事情就是通知.通知分为前置通知，后置通知，异常通知，最终通知，环绕通知（切面要完成的功能）
- **Aspect（切面）** 是切入点和通知（引介）的结合
- **Introduction（引介）** 引介是一种特殊的通知在不修改类代码的前提下， Introduction可以在运行期为类动态地添加一些方法或Field.
- **Target（目标对象）** 代理的目标对象（要增强的类）
- **Weaving（织入）** 是把增强应用到目标的过程，把advice 应用到 target的过程
- **Proxy（代理）** 一个类被AOP织入增强后，就产生一个结果代理类

AOP（Aspect Oriented Programming ）称为面向切面编程，扩展功能不是修改源代码实现，在程序开发中主要用来解决一些系统层面上的问题，比如日志，事务，权限等待，Struts2的拦截器设计就是基于AOP的思想，是个比较经典的例子。

## 7. Spring 是如何管理事务的，事务管理机制

事务管理可以帮助我们保证数据的一致性，对应企业的实际应用很重要。

Spring的事务机制包括声明式事务和编程式事务。

- **编程式事务管理**：Spring推荐使用TransactionTemplate，实际开发中使用声明式事务较多。
- **声明式事务管理**：将我们从复杂的事务处理中解脱出来，获取连接，关闭连接、事务提交、回滚、异常处理等这些操作都不用我们处理了，Spring都会帮我们处理。

**声明式事务管理使用了AOP面向切面编程实现的，本质就是在目标方法执行前后进行拦截。在目标方法执行前加入或创建一个事务，在执行方法执行后，根据实际情况选择提交或是回滚事务**。

### 如何管理的

Spring事务管理主要包括3个接口，Spring的事务主要是由它们(**PlatformTransactionManager，TransactionDefinition，TransactionStatus**)三个共同完成的。

**1. PlatformTransactionManager**：事务管理器–主要用于平台相关事务的管理

主要有三个方法：

- commit 事务提交；
- rollback 事务回滚；
- getTransaction 获取事务状态。

**2. TransactionDefinition**：事务定义信息–用来定义事务相关的属性，给事务管理器PlatformTransactionManager使用

这个接口有下面四个主要方法：

- getIsolationLevel：获取隔离级别；
- getPropagationBehavior：获取传播行为；
- getTimeout：获取超时时间；
- isReadOnly：是否只读（保存、更新、删除时属性变为false–可读写，查询时为true–只读）

事务管理器能够根据这个返回值进行优化，这些事务的配置信息，都可以通过配置文件进行配置。

**3. TransactionStatus**：事务具体运行状态–事务管理过程中，每个时间点事务的状态信息。

例如它的几个方法：

- hasSavepoint()：返回这个事务内部是否包含一个保存点，
- isCompleted()：返回该事务是否已完成，也就是说，是否已经提交或回滚
- isNewTransaction()：判断当前事务是否是一个新事务

**声明式事务的优缺点**：

- **优点**：不需要在业务逻辑代码中编写事务相关代码，只需要在配置文件配置或使用注解（@Transaction），这种方式没有侵入性。
- **缺点**：声明式事务的最细粒度作用于方法上，如果像代码块也有事务需求，只能变通下，将代码块变为方法。

## 8. Spring中bean加载机制，生命周期

### 加载机制

 [详解Spring中Bean的加载](https://www.cnblogs.com/weknow619/p/6673667.html)

### 生命周期

在传统的Java应用中，bean的生命周期很简单。使用Java关键字new进行bean实例化，然后该bean就可以使用了。一旦该bean不再被使用，则由Java自动进行垃圾回收。

相比之下，Spring容器中的bean的生命周期就显得相对复杂多了。正确理解Spring bean的生命周期非常重要，因为你或许要利用Spring提供的扩展点来自定义bean的创建过程。下图展示了bean装载到Spring应用上下文中的一个典型的生命周期过程。

[![img](https://github.com/orangehaswing/fullstack-tutorial/raw/master/notes/JavaArchitecture/assets/bean-life.png)](https://github.com/orangehaswing/fullstack-tutorial/blob/master/notes/JavaArchitecture/assets/bean-life.png)

上图bean在Spring容器中从创建到销毁经历了若干阶段，每一阶段都可以针对Spring如何管理bean进行个性化定制

**正如你所见，在bean准备就绪之前，bean工厂执行了若干启动步骤。我们对上图进行详细描述：**

1. Spring 对 Bean 进行实例化；
   - 相当于程序中的new Xx()
2. Spring 将值和 Bean 的引用注入进 Bean 对应的属性中；
3. **如果Bean实现了 BeanNameAware 接口**，Spring 将 Bean 的 ID 传递给setBeanName()方法
   - 实现BeanNameAware接口主要是为了通过Bean的引用来获得Bean的ID，一般业务中是很少有在Bean的ID的
4. **如果Bean实现了BeanFactoryAware接口**，Spring将调用setBeanDactory(BeanFactory bf)方法并把BeanFactory容器实例作为参数传入。
   - 实现BeanFactoryAware 主要目的是为了获取Spring容器，如Bean通过Spring容器发布事件等
5. **如果Bean实现了ApplicationContextAwaer接口**，Spring容器将调用setApplicationContext(ApplicationContext ctx)方法，将bean所在的应用上下文的引用传入进来
   - 作用与BeanFactory类似都是为了获取Spring容器，不同的是Spring容器在调用setApplicationContext方法时会把它自己作为setApplicationContext 的参数传入，而Spring容器在调用setBeanDactory前需要程序员自己指定（注入）setBeanDactory里的参数BeanFactory
6. **如果Bean实现了BeanPostProcess接口**，Spring将调用它们的postProcessBeforeInitialization（预初始化）方法
   - 作用是在Bean实例创建成功后对进行增强处理，如对Bean进行修改，增加某个功能
7. **如果Bean实现了InitializingBean接口**，Spring将调用它们的afterPropertiesSet方法，作用与在配置文件中对Bean使用init-method声明初始化的作用一样，都是在Bean的全部属性设置成功后执行的初始化方法。
8. **如果Bean实现了BeanPostProcess接口**，Spring将调用它们的postProcessAfterInitialization（后初始化）方法
   - 作用与6的一样，只不过6是在Bean初始化前执行的，而这个是在Bean初始化后执行的，时机不同
9. 经过以上的工作后，Bean将一直驻留在应用上下文中给应用使用，直到应用上下文被销毁
10. **如果Bean实现了DispostbleBean接口**，Spring将调用它的destory方法，作用与在配置文件中对Bean使用destory-method属性的作用一样，都是在Bean实例销毁前执行的方法。

## 9. Bean实例化的三种方式

- 使用类的无参构造创建（此种方式用的最多）
- 使用静态工厂创建对象
- 使用实例工厂创建对象

## 10. BeanFactory 和 FactoryBean的区别

- **BeanFactory**是个Factory，也就是IOC容器或对象工厂，在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的，提供了实例化对象和拿对象的功能。
- **FactoryBean**是个Bean，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,它的实现与设计模式中的工厂模式和修饰器模式类似。

## 11. BeanFactory和ApplicationContext的区别

### BeanFactory

是Spring里面最低层的接口，提供了最简单的容器的功能，只提供了实例化对象和拿对象的功能。

### 两者装载bean的区别

- **BeanFactory**：在启动的时候不会去实例化Bean，中有从容器中拿Bean的时候才会去实例化；
- **ApplicationContext**：在启动的时候就把所有的Bean全部实例化了。它还可以为Bean配置lazy-init=true来让Bean延迟实例化；

### 我们该用BeanFactory还是ApplicationContent

**BeanFactory** 延迟实例化的优点：

应用启动的时候占用资源很少，对资源要求较高的应用，比较有优势；

缺点：速度会相对来说慢一些。而且有可能会出现空指针异常的错误，而且通过bean工厂创建的bean生命周期会简单一些

**ApplicationContext** 不延迟实例化的优点：

- 所有的Bean在启动的时候都加载，系统运行的速度快；
- 在启动的时候所有的Bean都加载了，我们就能在系统启动的时候，尽早的发现系统中的配置问题
- 建议web应用，在启动的时候就把所有的Bean都加载了。

缺点：把费时的操作放到系统启动中完成，所有的对象都可以预加载，缺点就是消耗服务器的内存

### ApplicationContext其他特点

除了提供BeanFactory所支持的所有功能外，ApplicationContext还有额外的功能

- 默认初始化所有的Singleton，也可以通过配置取消预初始化。
- 继承MessageSource，因此支持国际化。
- 资源访问，比如访问URL和文件（ResourceLoader）；
- 事件机制，（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，比如应用的web层；
- 同时加载多个配置文件。
- 消息发送、响应机制（ApplicationEventPublisher）；
- 以声明式方式启动并创建Spring容器。

由于ApplicationContext会预先初始化所有的Singleton Bean，于是在系统创建前期会有较大的系统开销，但一旦ApplicationContext初始化完成，程序后面获取Singleton Bean实例时候将有较好的性能。

也可以为bean设置lazy-init属性为true，即Spring容器将不会预先初始化该bean。

### spring的AOP（常用的是拦截器）

一般拦截器都是实现HandlerInterceptor，其中有三个方法preHandle、postHandle、afterCompletion

1. preHandle：执行controller之前执行
2. postHandle：执行完controller，return modelAndView之前执行，主要操作modelAndView的值
3. afterCompletion：controller返回后执行

### spring载入多个上下文

不同项目使用不同分模块策略，spring配置文件分为

- applicationContext.xml(主文件，包括JDBC配置，hibernate.cfg.xml，与所有的Service与DAO基类)
- applicationContext-cache.xml(cache策略，包括hibernate的配置)
- applicationContext-jmx.xml(JMX，调试hibernate的cache性能)
- applicationContext-security.xml(acegi安全)
- applicationContext-transaction.xml(事务)
- moduleName-Service.xml
- moduleName-dao.xml

## 12. ApplicationContext 上下文的生命周期

PS：可以借鉴Servlet的生命周期，实例化、初始init、接收请求service、销毁destroy;

Spring上下文中的Bean也类似，【Spring上下文的生命周期】

1. 实例化一个Bean，也就是我们通常说的new；
2. 按照Spring上下文对实例化的Bean进行配置，也就是IOC注入
3. 如果这个Bean实现了BeanNameAware接口，会调用它实现的setBeanName(String beanId)方法，此处传递的是Spring配置文件中Bean的ID；
4. 如果这个Bean实现了BeanFactoryAware接口，会调用它实现的setBeanFactory()，传递的是Spring工厂本身（可以用这个方法获取到其他Bean）；
5. 如果这个Bean实现了ApplicationContextAware接口，会调用setApplicationContext(ApplicationContext)方法，传入Spring上下文，该方式同样可以实现步骤4，但比4更好，以为ApplicationContext是BeanFactory的子接口，有更多的实现方法；
6. 如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforeInitialization(Object obj, String s)方法，BeanPostProcessor经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用After方法，也可用于内存或缓存技术；
7. 如果这个Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法；
8. 如果这个Bean关联了BeanPostProcessor接口，将会调用postAfterInitialization(Object obj, String s)方法；

注意：以上工作完成以后就可以用这个Bean了，那这个Bean是一个single的，所以一般情况下我们调用同一个ID的Bean会是在内容地址相同的实例

1. 当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean接口，会调用其实现的destroy方法
2. 最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法

以上10步骤可以作为面试或者笔试的模板，另外这里描述的是应用Spring上下文Bean的生命周期，如果应用Spring的工厂也就是BeanFactory的话去掉第5步就Ok了；

## 13. Spring中autowire和resourse关键字的区别

@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是Spring的注解，它的包是javax.annotation.Resource，需要导入，但是Spring支持该注解的注入。

1、共同点

两者都可以写在字段和setter方法上。两者如果都写在字段上，那么就不需要再写setter方法。

2、不同点

**（1）@Autowired**

@Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired;只按照byType注入。

```
public class TestServiceImpl {
    // 下面两种@Autowired只要使用一种即可
    @Autowired
    private UserDao userDao; // 用于字段上
    
    @Autowired
    public void setUserDao(UserDao userDao) { // 用于属性的方法上
        this.userDao = userDao;
    }
}
```

@Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的required属性为false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。如下：

```
public class TestServiceImpl {
    @Autowired
    @Qualifier("userDao")
    private UserDao userDao; 
}
```

**（2）@Resource**

@Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。@Resource有两个重要的属性：name和type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。

```
public class TestServiceImpl {
    // 下面两种@Resource只要使用一种即可
    @Resource(name="userDao")
    private UserDao userDao; // 用于字段上
    
    @Resource(name="userDao")
    public void setUserDao(UserDao userDao) { // 用于属性的setter方法上
        this.userDao = userDao;
    }
}
```

注：最好是将@Resource放在setter方法上，因为这样更符合面向对象的思想，通过set、get去操作属性，而不是直接去操作属性。

@Resource装配顺序：

1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。
3. 如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。
4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

@Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。

## 14. Spring的注解讲一下，介绍Spring中的熟悉的注解

思考：spring怎么知道应该哪些Java类当成bean类处理？

答案：使用配置文件或者注解的方式进行标识需要处理的java类!

### 一： 组件类注解

@Component ：标准一个普通的spring Bean类。 @Repository：标注一个DAO组件类。 @Service：标注一个业务逻辑组件类。 @Controller：标注一个控制器组件类。

这些都是注解在平时的开发过程中出镜率极高，@Component、@Repository、@Service、@Controller实质上属于同一类注解，用法相同，功能相同，区别在于标识组件的类型。@Component可以代替@Repository、@Service、@Controller，因为这三个注解是被@Component标注的。如下代码

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    String value() default "";
}
```

举例：

（1）当一个组件代表数据访问层（DAO）的时候，我们使用@Repository进行注解，如下

```
@Repository
public class HappyDaoImpl implements HappyDao{
private final static Logger LOGGER = LoggerFactory.getLogger(HappyDaoImpl .class);
public void  club(){
        //do something ,like drinking and singing
    }
}1234567
```

（2）当一个组件代表业务层时，我们使用@Service进行注解，如下

```
@Service(value="goodClubService")
//使用@Service注解不加value ,默认名称是clubService
public class ClubServiceImpl implements ClubService {
    @Autowired
    private ClubDao clubDao;

    public void doHappy(){
        //do some Happy
    }
 }12345678910
```

（3）当一个组件作为前端交互的控制层，使用@Controller进行注解，如下

```
@Controller
public class HappyController {
    @Autowired //下面进行讲解
    private ClubService clubService;

    // Control the people entering the Club
    // do something
}
/*Controller相关的注解下面进行详细讲解，这里简单引入@Controller*/
```

**3、总结注意点**

1. 被注解的java类当做Bean实例，Bean实例的名称默认是Bean类的首字母小写，其他部分不变。@Service也可以自定义Bean名称，但是必须是唯一的！
2. 尽量使用对应组件注解的类替换@Component注解，在spring未来的版本中，@Controller，@Service，@Repository会携带更多语义。并且便于开发和维护！
3. 指定了某些类可作为Spring Bean类使用后，最好还需要让spring搜索指定路径，在Spring配置文件加入如下配置：

```
<!-- 自动扫描指定包及其子包下的所有Bean类 -->
<context:component-scan base-package="org.springframework.*"/>
```

### 二：装配bean时常用的注解

@Autowired：属于Spring 的org.springframework.beans.factory.annotation包下,可用于为类的属性、构造器、方法进行注值

 @Resource：不属于spring的注解，而是来自于JSR-250位于java.annotation包下，使用该annotation为目标bean指定协作者Bean。

## 15. Spring 中用到了那些设计模式？

Spring框架中使用到了大量的设计模式，下面列举了比较有代表性的：

- 代理模式—在AOP和remoting中被用的比较多。
- 单例模式—在spring配置文件中定义的bean默认为单例模式。
- 模板方法—用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。
- 工厂模式—BeanFactory用来创建对象的实例。
- 适配器–spring aop
- 装饰器–spring data hashmapper
- 观察者– spring 时间驱动模型
- 回调–Spring ResourceLoaderAware回调接口

### 工厂模式（Factory Method）

Spring容器就是实例化和管理Bean的工厂

工厂模式隐藏了创建类的细节，返回值必定是接口或者抽象类,而不是具体的某个对象，工厂类根据条件生成不同的子类实例。当得到子类的实例后，就可以调用基类中的方法，不必考虑返回的是哪一个子类的实例。

这个很明显，在各种BeanFactory以及ApplicationContext创建中都用到了；

**Spring通过配置文件，就可以管理所有的bean，而这些bean就是Spring工厂能产生的实例，因此，首先我们在Spring配置文件中对两个实例进行配置**。

### 单态模式【单例模式】（Singleton）

**Spring默认将所有的Bean设置成 单例模式，即对所有的相同id的Bean的请求，都将返回同一个共享的Bean实例。这样就可以大大降低Java创建对象和销毁时的系统开销**。

使用Spring将Bean设置称为单例行为，则无需自己完成单例模式。

| 可以通过singleton=“true | false” 或者 scope=“？”来指定 |

### 适配器（Adapter）

**在Spring的Aop中，使用的Advice（通知）来增强被代理类的功能。Spring实现这一AOP功能的原理就使用代理模式（1、JDK动态代理。2、CGLib字节码生成技术代理。）对类进行方法级别的切面增强，即，生成被代理类的代理类， 并在代理类的方法前，设置拦截器，通过执行拦截器重的内容增强了代理方法的功能，实现的面向切面编程**。

### 代理（Proxy）

Spring实现了一种能够通过额外的方法调用完成任务的设计模式 - 代理设计模式,比如JdkDynamicAopProxy和Cglib2AopProxy。

代理设计模式的一个很好的例子是**org.springframework.aop.framework.ProxyFactoryBean**。**该工厂根据Spring bean构建AOP代理。该类实现了定义getObject()方法的FactoryBean接口。此方法用于将需求Bean的实例返回给bean factory**。在这种情况下，它不是返回的实例，而是AOP代理。在执行代理对象的方法之前，可以通过调用补充方法来进一步“修饰”代理对象(其实所谓的静态代理不过是在装饰模式上加了个要不要你来干动作行为而已，而不是装饰模式什么也不做就加了件衣服，其他还得由你来全权完成)。

### 观察者（Observer）

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。**spring中Observer模式常用的地方是listener的实现。如ApplicationListener**。

补充面试题：Spring里面的工厂模式和代理模式，IO中的装饰者模式，挑几个最熟的能讲讲思路和伪代码实现？

## 16. Spring 的优点有哪些

1. 降低了组件之间的耦合性 ，实现了软件各层之间的解耦
2. 可以使用容易提供的众多服务，如事务管理，消息服务等
3. 容器提供单例模式支持
4. 容器提供了AOP技术，利用它很容易实现如权限拦截，运行期监控等功能
5. 容器提供了众多的辅助类，能加快应用的开发
6. spring对于主流的应用框架提供了集成支持，如hibernate，JPA，Struts等
7. spring属于低侵入式设计，代码的污染极低
8. 独立于各种应用服务器
9. spring的DI机制降低了业务对象替换的复杂性
10. Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可以自由选择spring的部分或全部

## 17. IOC和AOP用到的设计模式

用过spring的朋友都知道spring的强大和高深，都觉得深不可测，其实当你真正花些时间读一读源码就知道它的一些技术实现其实是建立在一些最基本的技术之上而已；例如AOP(面向方面编程)的实现是建立在CGLib提供的类代理和jdk提供的接口代理，IOC(控制反转)的实现建立在工厂模式、Java反射机制和jdk的操作XML的DOM解析方式。
