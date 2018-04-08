# Spring in Action
&emsp;&emsp;Spring 最根本的使命：简化Java开发。

&emsp;&emsp;为了降低Java开发的复杂性，Spring采取了以下4种关键策略：
+ 基于POJO的轻量级和最小侵入性编程；
+ 通过依赖注入和面向接口实现松耦合；
+ 基于切面和惯例进行声明式编程；
+ 通过切面和模板减少样板式代码。

&emsp;&emsp;Spring 竭力避免因自身的API而弄乱你的应用代码。Spring不会强迫你实现Spring规范的接口或继承Spring规范的类，相反，在基于Spring构建的应用中，它的类通常没有任何痕迹表明你使用了Spring。最坏的场景是，一个类或许会使用Spring注解，但它依旧是POJO。

&emsp;&emsp;Spring赋予POJO魔力的方式之一就是通过DI来装配它们。

#### 依赖注入
&emsp;&emsp;耦合具有两面性（two-headed beast）。一方面，紧密耦合的代码难以测试、难以复用、难以理解，并且典型地表现出“打地鼠”式的bug特性（修复一个bug，将会出现一个或者更多新的bug）。另一方面，一定程度的耦合又是必须的——完全没有耦合的代码什么也做不了。为了完成有实际意义的功能，不同的类必须以适当的方式进行交互。

&emsp;&emsp;通过DI，对象的依赖关系将由系统中负责协调各对象的第三方组件在创建对象的时候进行设定。对象无需自行创建或管理它们的依赖关系。

&emsp;&emsp;DI所带来的最大收益——松耦合。如果一个对象只通过接口（而不是具体实现或初始化过程）来表明依赖关系，那么这种依赖就能够在对象本身毫不知情的情况下，用不同的具体实现进行替换。

#### 应用切面
&emsp;&emsp;面向切面编程（aspect-oriented programming，AOP）允许你把遍布应用各处的功能分离出来形成可重用的组件。

&emsp;&emsp;面向切面编程往往被定义为促使软件系统实现关注点的分离一项技术。系统由许多不同的组件组成，每一个组件各负责一块特定功能。除了实现自身核心的功能之外，这些组件还经常承担着额外的职责。诸如日志、事务管理和安全这样的系统服务经常融入到自身具有核心业务逻辑的组件中去，这些系统服务通常被称为横切关注点，因为它们会跨越系统的多个组件。

&emsp;&emsp;如果将这些关注点分散到多个组件中去，你的代码将会带来双重的复杂性：
+ 实现系统关注点功能的代码将会重复出现在多个组件中。这意味着如果你要改变这些关注点的逻辑，必须修改各个模块中的相关实现。即使你把这些关注点抽象为一个独立的模块，其他模块只是调用它的方法，但方法的调用还是会重复出现在各个模块中。
+ 组件会因为那些与自身核心业务无关的代码而变得混乱。

&emsp;&emsp;AOP能够使这些服务模块化，并以声明的方式将它们应用到它们需要影响的组件中去。所造成的结果就是这些组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解涉及系统服务所带来复杂性。总之，AOP能够确保POJO的简单性。

#### 使用模板消除样板式代码
&emsp;&emsp;JDBC不是产生样板式代码的唯一场景。在许多编程场景中往往都会导致类似的样板式代码，JMS、JNDI和使用REST服务通常也涉及大量的重复代码。

&emsp;&emsp;Spring旨在通过模板封装来消除样板式代码。Spring的JdbcTemplate使得执行数据库操作时，避免传统的JDBC样板代码成为了可能。

### 容纳你的bean
&emsp;&emsp;在基于Spring的应用中，你的应用对象生存于Spring容器（container）中。Spring容器负责创建对象，装配它们，配置它们并管理它们的整个生命周期，从生存到死亡。

&emsp;&emsp;容器是Spring框架的核心。Spring容器使用DI管理构成应用的组件，它会创建相互协作的组件之间的关联。两种不同的类型：
+ bean工厂（由`org.springframework.beans.factory.BeanFactory`接口定义）是最简单的容器，提供基本的DI支持。
+ 应用上下文（由`org.springframework.context.ApplicationContext`接口定义）基于BeanFactory构建，并提供应用框架级别的服务。

#### 应用上下文
+ AnnotationConfigApplicationContext：从一个或多个基于Java的配置类中加载Spring应用上下文。
+ AnnotationConfigWebApplicationContext：从一个或多个基于Java的配置类中加载SpringWeb应用上下文。
+ ClassPathXmlApplicationContext：从类路径下的一个或多个XML配置文件中加载上下文定义，把应用上下文的定义文件作为类资源。
+ FileSystemXmlapplicationcontext：从文件系统下的一个或多个XML配置文件中加载上下文定义。
+ XmlWebApplicationContext：从Web应用下的一个或多个XML配置文件中加载上下文定义。

## 装配bean
> 创建应用对象之间协作关系的行为通常称为装配（`wiring`）。

**三种主要的装配机制：**

1. 在XML中进行显式配置
2. 在Java中进行显式配置
3. 隐式的bean发现机制和自动装配

### 自动化装配bean
Spring从两个角度来实现自动化装配：

+ 组件扫描（`component scanning`）：Spring会自动发现应用上下文中所创建的Bean。
+ 自动装配（`autowiring`）：Spring自动满足bean之间的依赖。

#### 组件扫描：
```java
package soundsystem;
// 压缩碟片接口
public interface CompactDisc {
  void play();
}
```
```java
package soundsystem;
import org.springframework.stereotype.Component;
// 某张专辑
@Component// 注解表明该类会作为组件类，并告知Spring要为这个类创建bean
public class SgtPeppers implements CompactDisc {

  private String title = "Sgt. Pepper's Lonely Hearts Club Band";  // 专辑名称
  private String artist = "The Beatles"; // 艺术家
  
  public void play() {
    System.out.println("Playing " + title + " by " + artist);
  }
  
}

```
```java
package soundsystem;
import org.springframework.context.annotation.componentScan;
import org.springframework.context.annotation.Configuration;
// 组件扫描默认是不启用的，需要显式配置一下Spring，从而命令它去寻找带有@Component注解的类
@Configuration
@ComponentScan // 注解能够在Spring中启用组件扫描。默认会扫描与配置类相同的包。
// sounsystem包以及这个包下的所有子包，查找带有@Component注解的类。
public class CDPlayerConfig {}
```
```xml
<!-- 使用XML来启用组件扫描 -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:c="http://www.springframework.org/schema/c"
  xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

  <context:component-scan base-package="soundsystem" />

</beans>
```

1. Spring应用上下文中所有的bean都会给定一个ID。没有明确设置，bean的ID就是将类名的第一个字母变为小写。设置ID的两种方式：
    + `@Component("lonelyHeartsClub")`
    + `@Named("lonelyHeartsClub")`，使用Java依赖注入规范（Java Dependency Injection）

2. 在`@ComponentScan`的`value`属性中指明要扫描的包的名称：
    `@ComponentScan("soundsystem")`
    `@ComponentScan(basePackages={"soundsystem", "video"})`
    `@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})`

    > 可以考虑在包中创建一个用来进行扫描的空标记接口（marker interface）。通过这种方式，你依然能够保持对重构友好的接口引用，但是可以避免引用任何实际的应用程序代码（在重构中这些应用代码有可能会从想要的扫描包中移除）

#### 自动装配：
```java
package soundsystem;
import org.springframework.beans.factory.annctation.Autowired;
import org.springframework.steteotype.Component;

@Component
public class CDPlayer implements MediaPlayer {
    private CompactDisc cd;

    @Autowired // 注解表明当Spring创建CDPlayerbean时，会通过此构造器来实例化并且会传入一个可设置给CompactDisc类型的bean
    public CDPlayer(CompactDisc cd) {
        this.cd = cd;
    }
}
```
```java

@Autowired // @Autowired注解不仅能够用在构造器上，还能用在属性的Setter方法上。
public void setCompactDisc(CompactDisc cd) {
    this.cd = cd;
}

@Autowired // Setter方法没有特殊之处。@Autowired注解可在类的任何方法上，作用完全相同
public void insertDisc(CompactDisc cd) {
    this.cd = cd;
}

@Inject // 可以用Java依赖注入规范的@Inject来替换@Autowired
public CDPlayer(CompactDisc cd) {
    this.cd = cd;
}
```
### 通过Java代码装配bean
> 有时候自动化配置的方案行不通，比如想要将第三方库中的组件装配到你的应用中，这种情况下是没有办法在它的类上添加`@Component`和`@Autowired`注解的。
> 通常会将`JavaConfig`放到单独的包中，使它与其他的应用程序逻辑分离开来，这样对于它的意图就不会产生困惑。

#### 声明简单的bean
```java
package soundsystem;
import org.springframework.coentxt.annotation.Configuration;

@Configuration // 注解表明这个类是个配置类，该类应该包含在Spring应用上下文中如何创建bean的细节
public class CDPlayConfig {

    @Bean // 注解将所产生的对象注册为Spring应用上下文中的Bean
    public CompactDisc sgtPeppers() {
        return new sgtPeppers();
    }
    // 这里是使用Java来进行描述的，因此我们可以发挥Java提供的所有功能，只要最终生成一个CompactDisc实例即可。

    // 默认情况下，bean的ID与带有@Bean注解的方法名一样。可以通过name属性指定过一个不同的名字：
    @Bean(name="lonelyHeartsClubBand")
}
```
#### 借助JavaConfig实现注入
```java
@Bean
public CDPlayer cdPlayer() {
    return new CDPlayer(sgtPeppers());
}
// 看起来，CompactDisc是通过调用sgtPeppers()得到的，但情况并非完全如此。
// 因为sgtPeppers()方法上添加了@Bean注解，Spring将会拦截所有对它的调用，
// 并确保直接返回该方法所创建的bean，而不是每次都对其进行实际的调用。默认情况下，Spring中的bean都是单例的。
```
通过调用方法来引用bean的方式有点令人困惑。还有一种理解起来更为简单的方式：
```java
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc) {
    return new CDPlayer(compactDisc);
}
// 当Spring调用cdPlayer()创建CDPlayerbean的时候，它会自动装配一个CompactDisc到配置方法中。
// 然后方法体就可以按照合适的方式来使用它。
```
### 通过XML装配bean
#### 声明简单的bean
> 在使用JavaConfig的时候，这意味着要创建一个带有`@Configuration`注解的类，而在XML配置中，这意味着要创建一个XML文件，并且要以<beans>元素为根。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
    <bean class="soundsystem.SgtPeppers" />
</beans>
```
因为没有明确给定ID，所以这个bean将会根据全限定类名来进行命名。在本例中ID将会是`soundsystem.SgtPeppers#0`。可以借助id属性为bean设置名字：
`<bean id="compactDisc" class="soundsystem.SgtPeppers" />`
#### 借助构造器注入初始化bean
1. **两种基本的配置方案可供选择**：
    * `<construcor-arg>`元素
```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer">
    <constructor-arg ref="compactDisc" />
</bean>
```
    * 使用Spring3.0所引入的c-命名空间
```xml
<!-- 要在XML的顶部声明其模式 -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">
<!-- 属性名以"c:"开头，也就是命名空间的前缀。接下来就是要装配的构造器参数名，在此之后是"-ref"。 -->
    <bean id="cdPlayer" class="soundsystem.CDPlayer" c:cd-ref="compactDisc" />
<!-- 替代方案：使用参数在整个参数列表中的位置信息 -->
    <bean id="cdPlayer" class="soundsystem.CDPlayer" c:_0-ref="compactDisc" />
<!-- 只有一个构造器参数时 -->
    <bean id="cdPlayer" class="soundsystem.cdPlayer" c:_-ref="compactDisc" />
</beans>
```
2. **将字面量注入到构造器中**
```java
public class BlankDisc implements CompactDisc {
    private String title;
    private String artist;

    public BlankDisc(String title, String artist) {
        this.title = title;
        this.artist = artist;
    }
}
```
```xml
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
    <constructor-arg value="The Beatles" />
</bean>

<!-- 使用c-命名空间 -->
<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      c:_title="Sgt. Pepper's Lonely Hearts Club Band"
      c:_artist="The Beatles" />
<!-- 参数索引 -->
<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      c:_0="Sgt. Pepper's Lonely Hearts Club Band"
      c:_1="The Beatles" />
<!-- 只有一个构造器参数 -->
<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      c:_="Sgt. Pepper's Lonely Hearts Club Band" />
```
3. **装配集合**
```java
public class BlankDisc implements CompactDisc {
    private String title;
    private List<String> tracks;
    private List<CompactDisc> cds;

    public BlankDisc(String title, List<String> tracks, List<CompactDisc> cds) {
        this.title = title;
        this.tracks = tracks;
        this.cds = cds;
    }
}
```
```xml
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
    <constructor-arg>
        <list>
            <value>String1</value>
            <value>String2</value>
            <value>String3</value>
        </list>
    </constructor-arg>
    <constructor-arg>
        <list>
            <ref bean="sgtPeppers" />
            <ref bean="whiteAlbum" />
            <ref bean="hradDaysNight" />
        </list>
    </constructor-arg>
</bean>
<!-- 目前，使用c-命名空间的属性无法实现装配集合的功能。 -->
```
#### 设置属性
```java
public class CDPlayer implements MediaPlayer {
    private CompactDisc compactDisc;

    @Autowired
    public void setCompactDisc(CompactDisc compactDisc) {
        this.compactDisc = compactDisc;
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-xsd">
    <bean id="cdPlayer" class="soundsystem.CDPlayer">
        <property name="compactDisc" ref="compactDisc" />
    </bean>
    <!-- p-命名空间方式 使用"p:"前缀，接下来就是要注入的属性名。最后以"-ref"结尾 -->
    <bean id="cdPlayer" class="soundsystem.CDPlayer" p:compactDisc-ref="compactDisc" />
</beans>
```
将字面量注入到属性中
```java
public class BlankDisc implements CompactDisc {
    private String title;
    private String artist;
    private List<String> tracks;

    public void setTitle(String title) {
        this.title = title;
    }
    public void setARtist(String artist) {
        this.artist = artist;
    }
    public void setTracks(List<String> tracks) {
        this.tracks = tracks;
    }
}
```
```xml
<bean id="compactDisc" class="soundsystem.BlankDisc">
    <property name="title" value="Sgt. Pepper's Lonely Hearts Club Band" />
    <property name="artist" value="The Beatles" />
    <property name="tracks">
        <list>
            <value>String1</value>
            <value>String2</value>
            <value>String3</value>
        </list>
    </property>
</bean>
<!-- p-命名空间方案 -->
<bean id="compactDisc"
      class="soundsystem.BlankDisc"
      p:title="Sgt. Pepper's Lonely Hearts Club Band"
      p:artist="The Beatles">
    <property name="tracks">
        <list>
            <value>String1</value>
            <value>String2</value>
            <value>String3</value>
        </list>
    </property>
</bean>
```

## 面向切面的Spring
**Spring提供了4种类型的AOP支持：**

+ 基于代理的经典Spring AOP
+ 纯POJO切面
+ `@AspectJ`注解驱动的切面
+ 注入式AspectJ切面（适用于Spring各版本）

前三种都是Spring AOP实现的变体，Spring AOP构建在动态代理基础之上，因此，Spring对AOP的支持局限于方法拦截。

**AOP术语**

+ **通知（Advice）**：切面的工作被称为通知，通知定义了切面是什么以及何时使用。除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。5种类型的通知：
    - 前置通知（`Before`）：在目标方法被调用前调用通知功能
    - 后置通知（`After`）：在目标方法完成之后调用通知，不会关系方法输出是什么
    - 返回通知（`After-returning`）：在目标方法成功执行之后调用通知
    - 异常通知（`After-throwing`）：在目标方法抛出异常后调用通知
    - 环绕通知（`Around`）：包裹了被通知的方法，在被通知方法调用之前和之后执行
+ **连接点（Join point）**：应用可能有数以千计的时机应用通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。
+ **切点（Poincut）**：如果说通知定义了切面的“什么“和”何时“的话，切么切点就定义了“何处”。切点的定义会匹配通知所要织入的一个或多个连接点。
+ **切面（Aspect）**：切面时通知和切点的结合。
+ **引入（Introduction）**：引入允许我们向现有的类添加新方法或属性。
+ **织入(Weaving)**：把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期里有多个点可以进行织入：
    * 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器。AspectJ的织入编译器就是以这种方式织入切面的。
    * 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器（ClassLoader），它可以在目标类被引入之前增强该目标类的字节码。AspectJ 5的加载时织入（load-time weaving，LTW）就支持以这种方式织入切面。
    * 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。

### 通过切点来选择连接点
Spring AOP使用AspectJ的切点表达式语言来定义切点。仅支持AspectJ切点指示器（pointcut designator）的一个子集。

AspectJ指示器|描述
-|-
`execution()`|用于匹配是连接点的执行方法
`arg()`|限制连接点匹配参数为指定类型的执行方法
`@args()`|限制连接点匹配参数由指定注解标注的执行方法
`this()`|限制连接点匹配AOP代理的bean引用为指定类型的类
`target`|限制连接点匹配目标对象为指定类型的类
`@target()`|限制连接点匹配特定的执行对象，这些对象对应的类要具有指定类型的注解
`within()`|限制连接点匹配指定的类型
`@within()`|限制连接点匹配指定注解所标注的类型（当使用Spring AOP时，方法定义在由指定的注解所标注的类里）
`@annotation`|限定匹配带有指定注解的连接点

#### 编写切点
```java
package concert;
public interface Performance { // 演出接口
    public void perform(); // 表演
}
```
**使用AspectJ切点表达式来选择`Performance`的`perform()`方法：**
`execution(* concert.Performance.perform(..))`
方法表达式以`*`号开始，表明我们不关心方法返回值的类型。然后指定了全限定类名和方法名。`(..)`表明切点要选择任意的`perform()`方法，无论该方法的入参是什么。

仅匹配`concert`包：
`execution(* concert.Performance.perform(..)) && within(concert.*))`

#### 在切点中选择bean
`execution(* concert.Performance.perform(..)) and bean('woodstock')`
`execution(* concert.Performance.perform(..)) and !bean('woodstock')`

可以使用`and` `or` `not`分别代替`&&` `||` `！`

### 使用注解创建切面
#### 定义切面（前置通知、后置通知）
> Spring使用AspectJ注解来申明通知方法

```java
package concert;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect // 注解表明这是一个切面，这是一个观看演出的切面
public class Audience {
    // 通过@Pointcut注解声明频繁使用的切点表达式
    @Pointcut("execution(** concert.Performance.perform(..))") // 定义命名切点
    public void performance() {}

    @Before("performance()") // 表演之前
    public void silenceCellPhones() {
        System.out.println("Silencing cell phones");
    }
    @Before("performance()") // 表演之前
    public void takeSeats() {
        System.out.println("Taking seats");
    }
    @AfterReturning("performance()") // 表演之后
    public void applause() {
        System.out.println("CLAP CLAP CLAP!!!");
    }
    @AfterThrowing("performance()") // 表演失败之后
    public void demandRefund() {
        System.out.println("Demanding a refund");
    }
}
```
在JavaConfig中启用AspectJ注解的自动代理
```java
package concert;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy // 启用AspectJ自动代理
@ComponentScan
public class ConcertConfig {
    @Bean
    publci Audience audience() {
        return new Audience();
    }
}
```
在XML中启用AspectJ自动代理
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">

  <context:component-scan base-packapge="concert" />

  <aop:aspectj-autoproxy />

  <bean class="concert.Audience" />

</beans>
```
#### 创建环绕通知
```java
package concert;
import org.aspectj.lang.ProceedingJoinPoint;
import org.sapectj.lang.annotation.Around;
import org.sapectj.lang.annotation.Aspect;
import org.sapectj.lang.annotation.Pointcut;

@Aspect
public class Audience {
    @Pointcut("execution(** concert.Performance.perform(..))")
    public void performance() {}

    @Around("performance()") // 环绕通知方法
    public void watchPerformance(ProceedingJoinPoint jp) {
        try {
            System.out.println("Silencing cell phones");
            System.out.println("Taking seats");
            jp.proceed(); // 通过它来调用被通知的方法
            //  如果不调用这个方法，通知会阻塞对被通知方法的调用
            System.out.println("CLAP CLAP CLAP!!!");
        } catch (Throwable e) {
            System.out.println("Demanding a refund");
        }
    }
}
```
#### 处理通知中的参数
```java
@Aspect
public class TrackCounter {
    private Map<Integer, Integer> trackCounts = new HashMap<Integer, Integer>();

    // args(trackNumber)，它表明传递给playTrack()方法的int类型参数也会传递到通知中去。
    // 参数的名称trackNumber也与切点方法签名中的参数相匹配。
    @Pointcut("execution(* soundsystem.CompactDisc.playTrack(int)) && args(trackNumber)")
    public void trackPlayed(int trackNumber) {}

    @Before("trackPlayed(trackNumber)")
    public void countTrack(int trackNumber) {
        int currentCount = getPlayCount(trackNumber);
        trackCounts.put(trackNumber, currentCount + 1);
    }

    public int getPlayCount(int trackNumber) {
        return trackCounts.containsKey(trackNumber) ? trackCounts.get(trackNumber) : 0;
    }
}
```
```java
@Configuration
@EnableAspectJAutoProxy
public class TrackCounterConfig {
    @Bean
    public CompactDisc sgtPeppers() {
        BlankDisc cd = new BlankDisc();
        cd.setTitle("Sgt. Pepper's Lonely Hearts Club Band");
        cd.setArtist("The Beatles");
        List<String> tracks = new ArrayList<String>();
        tracks.add("Sgt. Pepper's Lonely Hearts Club Band");
        tracks.add("With a Little Help from My Friends");
        tracks.add("Lucy in the Sky with Diamonds");
        tracks.add("Getting Better");
        tracks.add("Fixing a Hole");

        cd.setTracks(tracks);
        return cd;
    }

    @Bean
    public TrackCounter trackCounter() {
        return new TrackCounter();
    }
}
```
### 在XML中声明切面
#### 声明前置和后置通知
```java
package concert;

public class Audience {
    public void silenceCellPhones() {
        System.out.println("Silencing cell phones");
    }
    public void takeSeats() {
        System.out.println("Taking seats");
    }
    public void applause() {
        System.out.println("CLAP CLAP CLAP!!!");
    }
    public void demandRefund() {
        System.out.println("Demanding a refund");
    }
}
```
```xml
<aop:config>
  <aop:aspect ref="audience">
    <!-- 定义切点，如果想让定义的切点能够在多个切面使用，可以把该元素放在<aop:config>元素的范围内 -->
    <aop:pointcut id="performance" expression="execution(** concert.Performance.perform(..))" />

    <aop:before pointcut="execution(** concert.Performance.perform(..))" method="silenceCellPhones" />
    <aop:before pointcut="execution(** concert.Performance.perform(..))" method="takeSeats" />
    <!-- 引用切点 -->
    <aop:after-returning pointcut-ref="performance" method="applause" />
    <aop:after-throwing pointcut-ref="performance" method="demandRefund" />
  </aop:aspect>
</aop:config>
```
#### 声明环绕通知
```java
package concert;
import org.aspectj.lang.ProceedingJoinPoint;

public class Audience {
    public void watchPerformance(ProceedingJoinPoint jp) {
        try {
            System.out.println("Silencing cell phones");
            System.out.println("Taking seats");
            jp.proceed();
            System.out.println("CLAP CLAP CLAP!!!");
        } catch (Throwable e) {
            System.out.println("Demanding a refund");
        }
    }
}

```
```xml
<aop:config>
  <aop:sapect ref="audience">
    <aop:pointcut id="performance" expression="execution(** concert.Performance.perform(..))" />
    <aop:around pointcut-ref="performance" method="watchPerformance" />
  </aop:sapect>
</aop:config>
```
#### 为通知传递参数
```java
public class TrackCounter {
    private Map<Integer, Integer> trackCounts = new HashMap<Integer, Integer>();

    public void countTrack(int trackNumber) {
        int currentCount = getPlayCount(trackNumber);
        trackCounts.put(trackNumber, currentCount + 1);
    }

    public int getPlayCount(int trackNumber) {
        return trackCounts.containsKey(trackNumber) ? trackCounts.get(trackNumber) : 0;
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean id="trackCounter" class="soundsystem.TrackCounter" />
  <bean id="cd" class="sounsystem.BlankDisc">
    <property name="title" value="Sgt. Pepper's Lonely Hearts Club Band" />
    <property name="artist" value="The Beatles" />
    <property name="tracks">
      <list>
        <value>Sgt. Pepper's Lonely Hearts Club Band</value>
        <value>With a Little Help from My Friends</value>
        <value>Lucy in the Sky with Diamonds</value>
        <value>Getting Better</value>
        <value>Fixing a Hole</value>
      </list>
    </property>
  </bean>
  <aop:config>
    <aop:aspect ref="trackCounter">
      <aop:pointcut id="trackPlayed"
      expression="execution(* soundsystem.CompactDisc.playTrack(int)) and args(trackNumer)" />
      <aop:before pointcut-ref="trackPlayed" method="countTrack" />
    </aop:aspect>
  </aop:config>
</beans>
```
## Spring MVC
### Spring MVC起步
#### 跟踪Spring MVC的请求
1. 请求离开浏览器，到达`DispatchServlet`（前段控制器）。
2. `DispatcherServlet`查询一个或多个处理器映射（`handler mapping`），将请求发送给选中的控制器。
3. 控制器完成逻辑处理后，通常会产生一些信息（模型，`model`），将模型数据打包，并且标示出用于渲染输出的视图名，将这些信息发送回`DispatcherServlet`。
4. `DispatcherServlet`使用视图解析器（`view resolver`）来将逻辑视图名匹配为一个特定的试图实现。
5. 视图将使用模型数据渲染输出，通过响应对象传递给客户端。
#### 搭建Spring MVC

扩展`AbstractAnnotationConfigDispatcherServletInitializer`的任意类都会自动地配置`DispatcherServlet`和`Spring`应用上下文，`Spring`的应用上下文会位于应用程序的`Servlet`上下文之中。

在Spring Web应用中，通常会有另外一个应用上下文，是由`ContextLoaderListener`创建的。`DispatcherServlet`加载包含Web组件的bean，如控制器、视图解析器以及处理器映射，而`ContextLoaderListener`要加载应用中的其他bean。这些bean通常是驱动应用后端得中间层和数据层组件。

通过`AbstractAnnotationConfigDispatcherServletInitializer`来配置`DispatcherServlet`是传统`web.xml`方式的替代方案。只能部署到支持Servlet 3.0的服务器中（Tomcat 7或更高版本）。

    package spittr.config;
    import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;
    import spittr.web.WebConfig;
    public class SpitterWebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
      @Override
      protected String[] getServletMappings() { // 将DispatcherServlet映射到"/"
        return new String[] { "/" };
      }
      @Override
      protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
      }
      @Override
      protected Class<?>[] getServletConfigClasses() { // 指定配置类
        return new Class<?>[] { WebConfig.class };
      }
    }

``` java
package spittr.config;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableWebMvc // 启用Spring MVC
@ComponentScan("spitter.web") // 启用组件扫描
public class WebConfig extends WebMvcConfigurerAdapter {
    // 如果没有配置视图解析器，Spring默认使用BeanNameViewResolver
    @Bean
    public ViewResolver viewResolver() { // 配置JSP试图解析器
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        return resolver;
    }
    // 配置静态资源的处理
    @Override
    public void configureDefaultServletHandling (DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```
```java
package spittr.config;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@ComponentScan(basePackage={"spitter"}, excludeFilters={
    @Filter(type=FilterType.ANNOTATION, value=EnableWebMvc.class)
    })
public class RootConfig {}
```

### 编写控制器
``` java
package spittr.web;
import java.util.List;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller // 声明为一个控制器，基于@Component注解，组件扫描器会自动找到
@RequestMapping("/spittles") // 类级别请求处理，注解会应用到控制器的所有处理器方法上
public class SpittleController {
    private SpittleReposiotry spittleRepository;

    @Autowired
    public SpittleController(SpittleRepository spittleRepository) { // 注入SpittleRepository
        this.spittleRepository = spittleRepository;
    }

    @RequestMapping(method=RequestMethod.GET) // 处理GET请求
    public String spittles(Model model) {
        // 将spittle添加到模型中
        model.addAttribute(spittleRepository.findSpittles(Long.MAX_VALUE, 20));
        return "spittles"; // 返回视图名
    }
}
```
`@RequestMapping`的`value`属性能够接受一个String类型的数组：`@RequestMapping({"/", "/homepage"})`

`Model`实际上就是一个Map，它会传递给视图，这样数据就能渲染到客户端。当调用`addAtribute()`方法并且不指定key的时候，那么key会根据值的对象类型推断确定。在本例中，因为它是一个`List<Spittle>`，因此，键将会推断为`spittleList`。
显式声明模型的key：
`model.addAttribute("spittleList", spittleRepository.findSpittles(Long.MAX_VALUE, 20));`
可替代方案：
```java
// 逻辑视图的名称将会根据请求路径推断得出。因为这个方法处理针对“/spittles”的GET请求，因此视图的名称将会是spittles
@RequestMapping(method=ReqeustMethod.GET)
public List<Spittle> spittles() {
    return spittleRepository.findSpittles(Long.MAX_VALUE, 20));
}
```
### 接受请求的输入
Spring MVC允许以多种方式将客户端中的数据传送到控制器的处理器方法中：

* 查询参数（Query Parameter）
* 表单参数（Form Parameter）
* 路径变量（Path Variable）

#### 处理查询参数
``` java
// /spittles?max=238900&count=50
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(@RequestParam("max") long max, @RequestParam("count") int count) {
    return spittleRepository.findSpittles(max, count);
}
```
同时处理有参数和没有参数的场景：
``` java
// 查询参数都是String类型的，因此defaultValue属性需要String类型的值。
private static final String MAX_LONG_AS_STRING = "9223372036854775807";

@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(
    @RequestParam(value="max", defaultValue=MAX_LONG_AS_STRING) long max, 
    @RequestParam(value="count", defaultValue="20") int count) {
    return spittleRepository.findSpittles(max, count);
}
```
#### 通过路径参数接受输入
面向资源的角度看。在理想情况下，要识别的资源应该通过URL路径进行标示，而不是通过查询参数。对"`/spittles/12345`"发起GET请求要优于对"`/spittles/show?spittle_id=12345`"发起请求。

为了实现这种路径变量 ，Spring MVC允许我们在`@RequestMapping`路径中添加占位符，用大括号括起来。路径中的其他部分要与所处理的请求完全匹配，但是占位符部分可以是任意的值。
```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle (@PathVariable("spittleId") long spittleId, Model model) {
    model.addAttribute(spittleRepository.findOne(spittleId));
    return "spittle";
}
```
因为方法的参数名碰巧与占位符的名称相同，因此我们可以去掉其中的value属性：
```java
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(@PathVariable long spittleId, Model model) {
    model.addAttribute(spittleRepository.findOne(spittleId));
    return "spittle";
}
```

### 处理表单
#### 编写处理表单的控制器
```java
@Controller
@RequestMapping("/spitter")
public class SpitterController {
    private SpitterRepository spitterRepository;

    @Autowired
    public SpitterController(SpitterRepository spitterRepository) {
        this.spitterRepository = spitterRepository;
    }

    @RequestMapping(value="/register", method=GET)
    public String showRegistrationForm() {
        return "registerForm";
    }

    @RequestMapping(value="/register", method=POST)
    public String processRegistration(Spitter spitter) {
        spitterRepository.save(spitter);
        return "redirect:/spitter/" + spitter.getUsername(); // 重定向到基本信息页
        // InternalResourceViewResolver能识别"redirect:", "forward:"前缀。
    }

    @RequestMapping(value="/{username}", method=GET)
    public String showSpitterProfile(@PathVariable String username, Model model) {
        Spitter spitter = spitterRepository.findByUesrname(username);
        model.addAttribute(spitter);
        return "profile";
    }
}
```

#### 校验表单
Java校验API
```java
package spittr;
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;

public class Spitter {

    private Long id;

    @NotNul
    @Size(min=5, max=16)
    private String username;

    @NotNul
    @Size(min=5, max=25)
    private String password;
}
```
```java
@RequestMapping(value="/register", method=POST)
public String processRegistration(@Valid Spitter spitter, Errors errors) {
    if (errors.hasErrors()) {
        return "registerForm";
    }
    spitterRepository.save(spitter);
    return "redirect:/spitter/" + spitter.getUsername();
}
```
