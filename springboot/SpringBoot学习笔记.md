# SpringBoot学习笔记

# $\color{red}**自动配置实现**$

### 一、概述

#### 1.主启动类@SpringBootApplication

​	主启动类的注解@SpringBootApplication继承了

> @SpringBootConfiguration
>
> > SpringBootConfiguration下继承了@Configuration，表示这是一个配置类
>
> @EnableAutoConfiguration***

​	@EnableAutoConfiguration是自动配置的关键

$\color{red}@EnableAutoConfiguration注解导入了AutoConfigurationImportSelector类，$

$\color{red}该类通过各种方法读取spring.factories下的内容，获得一系列自动配置类的集合$

#### 2.自动配置类

​	读取到的配置类都是在一定条件下才会生效的，

​	这一系列条件的判定通过spring5的注解@Conditional以及它的一系列派生注解实现

| 注解                            | 生效条件                                                   |
| ------------------------------- | ---------------------------------------------------------- |
| @ConditionalOnJava              | 应用使用指定的 Java 版本时生效                             |
| @ConditionalOnBean              | 容器中存在指定的 Bean 时生效                               |
| @ConditionalOnMissingBean       | 容器中不存在指定的 Bean 时生效                             |
| @ConditionalOnExpression        | 满足指定的 SpEL 表达式时生效                               |
| @ConditionalOnClass             | 存在指定的类时生效                                         |
| @ConditionalOnMissingClass      | 不存在指定的类时生效                                       |
| @ConditionalOnSingleCandidate   | 容器中只存在一个指定的 Bean 或这个 Bean 为首选 Bean 时生效 |
| @ConditionalOnProperty          | 系统中指定属性存在指定的值时生效                           |
| @ConditionalOnResource          | 类路径下存在指定的资源文件时生效                           |
| @ConditionalOnWebApplication    | 当前应用是 web 应用时生效                                  |
| @ConditionalOnNotWebApplication | 当前应用不是 web 应用生效                                  |

​	==当自动配置类所要求的条件都满足后，它便会引入它所需要的一些类（@Import）==

==并且将它的Properties类激活（@EnableConfigurationProperties），注入到IOC容器中，然后自动配置类再导入这些（个）Properties类，通过它的Properties类的参数要求去配置组件==

#### 3.Properties类

​	Properties类封装了自动配置组件时该组件配置的参数，这些类的属性注入再springboot的配置文件中（application.yaml、application.properties等），通过@ConfigurationProperties注解注入。

### 二、细节

#### 1.Spring Factories 机制

​	Spring Factories 机制是 Spring Boot 中的一种服务发现机制，Spring Boot 会自动扫描所有 Jar 包类路径下 META-INF/spring.factories 文件，并读取其中的内容，进行实例化，这种机制也是 Spring Boot Starter 的基础。

#### 2.Spring Factories 实现原理

​	spring-core 包里定义了 SpringFactoriesLoader 类，这个类会扫描所有 Jar 包类路径下的 META-INF/spring.factories 文件，并获取指定接口的配置。==通过这种方式，便可以从后续的自动配置包下的spring.fatories文件中读取其中的自动配置类的实例了。==

#### 3.autoconfigure下的spring.factories

​	在springboot项目下的spring-boot-autoconfigure-xxx.jar下的META-INF下有个叫spring.factories的文件，其本质与properties文件差不多。其中设置了springboot自动配置的内容。

​	该文件的就是键值对，键是一套成体系的实现框架（如：数据源的初始化环境等），值为一堆为支撑这个体系而存在的一系列自动配置类（如：web自动配置类、jdbc自动配置类等）。

### 三、写自己的自动配置类

#### 1.   概述

​		基于以上理论，我们也可以写一个自己的自动配置类。

​		简述为几个部分：1. AutoConfiguration类  2. Properties配置参数类  3.  要注入到容器中的实际功能类

#### 2.   开始实现

​		比如，我想要自动配置一个请求第三方接口的封装类，将其注入到容器中，该怎么做？

##### Properties类：

​		![image-20220708165228016](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708165228016.png)

​		可以使用@ConfigurationProperties指定配置前缀，从配置文件中读取信息注入到该Properties类中。

##### 实际功能类：

![image-20220708165522271](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708165522271.png)

##### 自动配置AutoConfiguration类：

![image-20220708165815454](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708165815454.png)

@AutoConfiguration表示这是一个自动配置类，它跟@Configuration差不多（实际上用@Configuration也可以），只不过是多了@AutoConfigureBefore和@AutoConfigureAfter的功能

##### 写spring.factories

从理论我们知道，springboot会读取所有META-INF下的spring.factories进行自动配置，因此，我们可以在我们项目的类路径下建立一个META-INF/spring.factories，然后将我们的配置类写到里面：

![image-20220708170816087](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708170816087.png)

![image-20220708170839516](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708170839516.png)

# $\color{red}**Spring核心**$

## 一、Spring对Bean的描述——BeanDenifition

### 1.   概述

​		BeanDenifition是Spring用来描述一个bean的接口，包括bean的属性、构造方法参数、依赖的 Bean 名称及是否单例、延迟加载等。

​		Spring就是根据BeanDefinition中的信息实例化Bean的。

​		BeanDefinition 整体可以分为两类，==一类是描述通用的 Bean，还有一类是描述注解形式的 Bean==。一般前者在 XML 时期定义  <bean> 标签以及在 Spring 内部使用较多，而现今我们大都使用后者，通过注解形式加载 Bean。

​		两类BeanDenifition各有各的实现类，在这里说一下使用注解方式的实现类：

> 1. AnnotatedGenericBeanDefinition：
>
>    该类继承自 GenericBeanDefinition ，并实现了 AnnotatedBeanDefinition 接口。==这个 BeanDefinition 用来描述标注 @Configuration 注解的 Bean==。
>
> 2. ScannedGenericBeanDefinition：
>
>    该类继承自 GenericBeanDefinition ，并实现了 AnnotatedBeanDefinition 接口。==这个  BeanDefinition 用来描述标注 @Component 注解的 Bean==，其派生注解如 @Service、@Controller  也同理。

### 2.   AbstractBeanDefinition

​		AbstractBeanDefinition 是 BeanDefinition 的子抽象类，也是其他 BeanDefinition  类型的基类，其实现了接口中定义的一系列操作方法，并定义了一系列的常量属性，这些常量会直接影响到 Spring 实例化 Bean 时的策略。

```java
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor
		implements BeanDefinition, Cloneable {

    // 默认的 SCOPE，默认是单例
	public static final String SCOPE_DEFAULT = "";

	// 不进行自动装配
	public static final int AUTOWIRE_NO = AutowireCapableBeanFactory.AUTOWIRE_NO;
    // 根据 Bean 的名字进行自动装配，byName
	public static final int AUTOWIRE_BY_NAME = AutowireCapableBeanFactory.AUTOWIRE_BY_NAME;
	// 根据 Bean 的类型进行自动装配，byType
	public static final int AUTOWIRE_BY_TYPE = AutowireCapableBeanFactory.AUTOWIRE_BY_TYPE;
	// 根据构造器进行自动装配
	public static final int AUTOWIRE_CONSTRUCTOR = AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR;
	// 首先尝试按构造器自动装配。如果失败，再尝试使用 byType 进行自动装配。（Spring 3.0 之后已废除）
	public static final int AUTOWIRE_AUTODETECT = AutowireCapableBeanFactory.AUTOWIRE_AUTODETECT;

    // 通过依赖检查来查看 Bean 的每个属性是否都设置完成
    // 以下常量分别对应：不检查、对依赖对象检查、对基本类型，字符串和集合进行检查、对全部属性进行检查
	public static final int DEPENDENCY_CHECK_NONE = 0;
	public static final int DEPENDENCY_CHECK_OBJECTS = 1;
	public static final int DEPENDENCY_CHECK_SIMPLE = 2;
	public static final int DEPENDENCY_CHECK_ALL = 3;

	// 关闭应用上下文时需调用的方法名称
	public static final String INFER_METHOD = "(inferred)";

    // 存放 Bean 的 Class 对象
	private volatile Object beanClass;

	// Bean 的作用范围
	private String scope = SCOPE_DEFAULT;

    // 非抽象
	private boolean abstractFlag = false;
	// 非延迟加载
	private boolean lazyInit = false;
    // 默认不自动装配
	private int autowireMode = AUTOWIRE_NO;
    // 默认不依赖检查
	private int dependencyCheck = DEPENDENCY_CHECK_NONE;

	// 依赖的 Bean 列表
	private String[] dependsOn;

    // 可以作为自动装配的候选者，意味着可以自动装配到其他 Bean 的某个属性中
	private boolean autowireCandidate = true;
    
	// 创建当前 Bean 实例工厂类名称
	private String factoryBeanName;
    // 创建当前 Bean 实例工厂类中方法名称
	private String factoryMethodName;

	// 存储构造方法的参数
	private ConstructorArgumentValues constructorArgumentValues;
    // 存储 Bean 属性名称以及对应的值
	private MutablePropertyValues propertyValues;
    // 存储被覆盖的方法信息
	private MethodOverrides methodOverrides;

	// init、destroy 方法名称
	private String initMethodName;
	private String destroyMethodName;

    // 是否执行 init 和 destroy 方法
	private boolean enforceInitMethod = true;
	private boolean enforceDestroyMethod = true;

    // Bean 是否是用户定义的而不是应用程序本身定义的
	private boolean synthetic = false;

    // Bean 的身份类别，默认是用户定义的 Bean
	private int role = BeanDefinition.ROLE_APPLICATION;

	// Bean 的描述信息
	private String description;

	// Bean 定义的资源
	private Resource resource;
	
	...
}
```



### 3.   AnnotatedBeanDefinition 	

​		在这里，AnnotatedBeanDefinition 是 BeanDefinition 子接口之一，该接口扩展了 BeanDefinition  的功能，其用来操作注解元数据。==一般情况下，通过注解方式得到的 Bean（@Component、@Bean），其 BeanDefinition  类型都是该接口的实现类==。

### 4.   ChildBeanDefinition

​		该类继承自 AbstractBeanDefinition。其相当于一个子类，不可以单独存在，必须依赖一个父 BeanDetintion，构造  ChildBeanDefinition 时，通过构造方法传入父 BeanDetintion 的名称或通过 setParentName  设置父名称。它可以从父类继承方法参数、属性值，并可以重写父类的方法，同时也可以增加新的属性或者方法。若重新定义 init 方法，destroy  方法或者静态工厂方法，ChildBeanDefinition 会重写父类的设置。而GenericBeanDefinition是 ChildBeanDefinition 更好的替代者。

### 5.   ConfigurationClassBeanDefinition

该类继承自 RootBeanDefinition（继承自AbstractBeanDefinition，在其基础上定义了更多属性） ，并实现了 AnnotatedBeanDefinition 接口。==这个 BeanDefinition 用来描述在标注 @Configuration 注解的类中==，通过 @Bean 注解实例化的 Bean。

其功能特点如下：

1. 如果 @Bean 注解没有指定 Bean 的名字，默认会用方法的名字命名 Bean。

2. ==标注 @Configuration 注解的类会成为一个工厂类，而标注 @Bean 注解的方法会成为工厂方法==，通过工厂方法实例化 Bean，而不是直接通过构造方法初始化。

3. ==标注 @Bean 注解的类会使用构造方法自动装配==

# $\color{red}**Web**$

### 一、静态资源导入

#### （1）webjars导入静态资源

​		在WebMvcAutoConfiguration下的addResourceHandlers方法下规定了webjars导入静态资源的规则。

​		他会在所有classpath:/META-INF/resources/webjars/下导入静态资源，这种方式是将静态资源打成jar包，然后供springboot扫描导入。

​		以这种方式访问静态资源时，可以：

​		localhost:8080/webjars/  +  静态资源

​		如：需要访问jquery的静态资源：

​		localhost:8080/webjars/jquery/3.4.1/jquery.js

![image-20220707170028806](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707170028806.png)

​	原理：方法规定了资源在浏览器url的映射

浏览器中localhost:8080/webjars/ 就映射到了资源下的classpath:/META-INF/resources/webjars/

#### （2）特定路径下导入静态资源（主要方法）

addResourceHandlers规定了另一种方法

![image-20220707170703341](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707170703341.png)

方法第二个参数this.mvcProperties.getStaticPathPattern()，获得路径 /**，表示项目路径下可以扫描到静态资源

在addResourceLocations中，this.resourceProperties.getStaticLocations()，添加了静态资源路径，返回下图的路径：

![image-20220707171139076](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707171139076.png)

总结：可以看出，在classpath:/resources、classpath:/META-INF/resources/、classpath:/static/、classpath:/public/、/**下都能扫到静态资源。

优先级：classpath:/resources  最高1

​				classpath:/static/  2

​				classpath:/public/  3

### 二、首页与图标

#### 1. 图标

##### （1）默认图标

> ```yaml
> spring.mvc.favicon.enabled=false ## 关闭
> ```

但在Spring Boot项目的issues中提出，如果提供默认的Favicon可能会导致网站信息泄露。如果用户不进行自定义的Favicon的设置，而Spring Boot项目会提供默认的上图图标，那么势必会导致泄露网站的开发框架。

==因此，在Spring Boot2.2.x中，将默认的favicon.ico移除==，同时也不再提供上述application.properties中的属性配置。

##### （2）自定义图标

正常情况下，直接将命名为favicon.ico的网站图标放在resources或static目录即可显示。

#### 2. 首页

* 首页名称为index，可以放到静态资源目录下，就会自动扫描。

### 三、Thymeleaf模板引擎

#### 1. 引入依赖 

引入最新版thymeleaf：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

如果不使用starter引入，而是引入一般性质的包的话，则需要注意版本问题。

在springboot 2.1.x版本中，thymeleaf默认支持是3.x的版本，如果引入2.x版本的thymeleaf是用不了的。

![image-20220707211431858](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707211431858.png)

依赖项说明在spring-boot-starter-parent下的spring-boot-dependencies（artifactId）中。

#### 2.  说明

在Thymeleaf的Properties中，可以看到模板引擎的前后缀：

![image-20220707212052150](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707212052150.png)

前缀为：classpath:/templates/

后缀为：.html

#### 3.  语法

##### （1）基本说明

项目所有的静态资源都应被thymeleaf接管 （@{}）

Thymeleaf 模板引擎支持多种表达式：

- 变量表达式：${...}
- 选择变量表达式：*{...}
- 链接表达式：@{...}
- 国际化表达式：#{...}
- 片段引用表达式：~{...}

##### （2）获取对象属性与方法

获取对象的属性和方法

使用变量表达式可以获取对象的属性和方法，例如，获取 person 对象的 lastName 属性，表达式形式如下：

```html
纯文本复制
${person.lastName}
```

##### （3）内置对象

使用变量表达式还可以使用内置基本对象，获取内置对象的属性，调用内置对象的方法。 Thymeleaf 中常用的内置基本对象如下：

- \#ctx ：上下文对象；
- \#vars ：上下文变量；
- \#locale：上下文的语言环境；
- \#request：HttpServletRequest 对象（仅在 Web 应用中可用）；
- \#response：HttpServletResponse 对象（仅在 Web 应用中可用）；
- \#session：HttpSession 对象（仅在 Web 应用中可用）；
- \#servletContext：ServletContext 对象（仅在 Web 应用中可用）。



例如，我们通过以下 2 种形式，都可以获取到 session 对象中的 map 属性：

```html
${#session.getAttribute('map')}
${session.map}
```

##### （4）内置工具对象

除了能使用内置的基本对象外，变量表达式还可以使用一些内置的工具对象。

- strings：字符串工具对象，常用方法有：equals、equalsIgnoreCase、length、trim、toUpperCase、toLowerCase、indexOf、substring、replace、startsWith、endsWith，contains 和 containsIgnoreCase 等；
- numbers：数字工具对象，常用的方法有：formatDecimal 等；
- bools：布尔工具对象，常用的方法有：isTrue 和 isFalse 等；
- arrays：数组工具对象，常用的方法有：toArray、length、isEmpty、contains 和 containsAll 等；
- lists/sets：List/Set 集合工具对象，常用的方法有：toList、size、isEmpty、contains、containsAll 和 sort 等；
- maps：Map 集合工具对象，常用的方法有：size、isEmpty、containsKey 和 containsValue 等；
- dates：日期工具对象，常用的方法有：format、year、month、hour 和 createNow 等。

例如，我们可以使用内置工具对象 strings 的 equals 方法，来判断字符串与对象的某个属性是否相等，代码如下。

```java
${#strings.equals('编程帮',name)}
```

##### （5）选择变量表达式

选择变量表达式与变量表达式功能基本一致，只是在变量表达式的基础上增加了与 th:object 的配合使用。当使用 th:object 存储一个对象后，我们可以在其后代中使用选择变量表达式（*{...}）获取该对象中的属性，其中，“*”即代表该对象。

```html
<div th:object="${session.user}" >    
    <p th:text="*{fisrtName}">firstname</p>
</div>
```

##### （6）链接表达式

不管是静态资源的引用，还是 form 表单的请求，凡是链接都可以用链接表达式 （@{...}）。

 链接表达式的形式结构如下：

- 无参请求：@{/xxx}
- 有参请求：@{/xxx(k1=v1,k2=v2)}

 例如使用链接表达式引入 css 样式表，代码如下。

```html
<link href="asserts/css/signin.css" th:href="@{/asserts/css/signin.css}" rel="stylesheet">
```

##### （7）国际化表达式

消息表达式一般用于国际化的场景。结构如下。

```html
th:text="#{msg}"
```

##### （8）片段引用表达式

片段引用表达式用于在模板页面中引用其他的模板片段（相当与vue组件），该表达式支持以下 2 中语法结构：

- 推荐：~{templatename::fragmentname}
- 支持：~{templatename::#id}

定义片段：th:fragment

引用片段：th:replace或th:insert

以上语法结构说明如下：

- templatename：模版名，Thymeleaf 会根据模版名解析完整路径：/resources/templates/templatename.html，要注意文件的路径。
- fragmentname：片段名，Thymeleaf 通过 th:fragment 声明定义代码块，即：th:fragment="fragmentname"
- id：HTML 的 id 选择器，使用时要在前面加上 # 号，不支持 class 选择器。

#### 4.  th属性

Thymeleaf 还提供了大量的 th 属性，这些属性可以直接在 HTML 标签中使用，其中常用 th 属性及其示例如下表。

 

| 属性        | 描述                                                         | 示例                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| th:id       | 替换 HTML 的 id 属性                                         | <input  id="html-id"    th:id="thymeleaf-id"/>               |
| th:text     | 文本替换，转义特殊字符                                       | <h1 th:text="hello，bianchengbang" >hello</h1>`              |
| th:utext    | 文本替换，不转义特殊字符                                     | <div th:utext="'<h1>欢迎来到编程帮！</h1>'" >欢迎你</div>    |
| th:object   | 在父标签选择对象，子标签使用 *{…} 选择表达式选取值。  没有选择对象，那子标签使用选择表达式和 ${…} 变量表达式是一样的效果。  同时即使选择了对象，子标签仍然可以使用变量表达式。 | <div th:object="${session.user}" >    <p th:text="*{fisrtName}">firstname</p></div> |
| th:value    | 替换 value 属性                                              | <input th:value = "${user.name}" />                          |
| th:with     | 局部变量赋值运算                                             | <div th:with="isEvens = ${prodStat.count}%2 == 0"  th:text="${isEvens}"></div> |
| th:style    | 设置样式                                                     | <div th:style="'color:#F00; font-weight:bold'">编程帮 www.biancheng.net</div> |
| th:onclick  | 点击事件                                                     | <td th:onclick = "'getInfo()'"></td>                         |
| th:each     | 遍历，支持 Iterable、Map、数组等。                           | <table>    <tr th:each="m:${session.map}">        <td th:text="${m.getKey()}"></td>        <td th:text="${m.getValue()}"></td>    </tr></table> |
| th:if       | 根据条件判断是否需要展示此标签                               | <a th:if ="${userId == collect.userId}">                     |
| th:unless   | 和 th:if 判断相反，满足条件时不显示                          | <div th:unless="${m.getKey()=='name'}" ></div>               |
| th:switch   | 与 Java 的 switch case语句类似  通常与 th:case 配合使用，根据不同的条件展示不同的内容 | <div th:switch="${name}">    <span th:case="a">编程帮</span>    <span th:case="b">www.biancheng.net</span></div> |
| th:fragment | 模板布局，类似 JSP 的 tag，用来定义一段被引用或包含的模板片段 | <footer th:fragment="footer">插入的内容</footer>             |
| th:insert   | 布局标签；  将使用 th:fragment 属性指定的模板片段（包含标签）插入到当前标签中。 | <div th:insert="commons/bar::footer"></div>                  |
| th:replace  | 布局标签；  使用 th:fragment 属性指定的模板片段（包含标签）替换当前整个标签。 | <div th:replace="commons/bar::footer"></div>                 |
| th:selected | select 选择框选中                                            | <select>    <option>---</option>    <option th:selected="${name=='a'}">        编程帮    </option>    <option th:selected="${name=='b'}">        www.biancheng.net    </option></select> |
| th:src      | 替换 HTML 中的 src 属性                                      | <img  th:src="@{/asserts/img/bootstrap-solid.svg}" src="asserts/img/bootstrap-solid.svg" /> |
| th:inline   | 内联属性；  该属性有 text、none、javascript 三种取值，  在 <script> 标签中使用时，js 代码中可以获取到后台传递页面的对象。 | <script type="text/javascript" th:inline="javascript">    var name = /*[[${name}]]*/ 'bianchengbang';    alert(name)</script> |
| th:action   | 替换表单提交地址                                             | <form th:action="@{/user/login}" th:method="post"></form>    |

### 四、  拓展配置MVC

如果想要保持springboot-mvc的特性并且扩展一些自定义功能的话，你可以在==WebMvcConfigurer==（接口）上添加@Configuration，==并且不添加@EnableWebMvc==（如果添加此注解，SpringMvc所有功能都会被接管）实现。

#### 1.   springmvc默认视图解析器

继承了springmvc提供的接口ViewResolver的类都可以被当作视图解析器。

springmvc提供了默认的视图解析器，ContentNegotiatingViewResolver，该类在Controller进行视图转发时，判断候选视图，并找出最佳视图进行转发。

该过程在这个类的resolveViewName方法下：

![image-20220707214123281](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707214123281.png)

#### 2.   springmvc如何获取候选视图？

上图可以看到，获取候选视图的方法：getCandidateViews：

![image-20220707215133263](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707215133263.png)

#### 3.   自定义视图解析器

基于以上理论，我们可以自己写一个视图解析器：

![image-20220707220210753](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220707220210753.png)

对于springMVC的拓展，如果用户有自定义的bean就用用户的，没有就用默认的。

#### 4.   为什么不能添加@EnableWebMvc呢？

首先我们先看一下WebMvcAutoConfiguration的配置条件

![image-20220708161113729](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708161113729.png)

再来看看@EnableWebMvc干了什么：

![image-20220708161351979](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708161351979.png)

很明显，这个注解导入了DelegatingWebMvcConfiguration这个类，而这个类继承自WebMvcAutoConfiguration，所有添加这个注解后，WebMvc的自动配置类失效。

![image-20220708161528201](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220708161528201.png)

#### 5.   配置国际化解析

##### （1）使用

* 可以在resource下建立国际化的资源包i18n
* 可根据页面提供多种语言，如登录页面可以创建几种语言源：

				1. 默认login.properties
				1. 中国：login_zh_CN.properties
				1. 英美：login_en_US.properties

##### （2）配置生效探究

这里又涉及到一个类：==MessageSourceAutoConfiguration==

里面有一个方法，这里发现SpringBoot已经自动配置好了管理国际化资源文件的组件 ResourceBundleMessageSource，==这一个组件的主要作用是读取国际化资源文件的基本名（basename）以及编码==：

```java
// 获取 properties 传递过来的值进行判断
// 给后续LocaleResolver映射资源文件提供basename
@Bean
public MessageSource messageSource(MessageSourceProperties properties) {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    if (StringUtils.hasText(properties.getBasename())) {
        // 设置国际化文件的基础名（去掉语言国家代码的）
        messageSource.setBasenames(
            StringUtils.commaDelimitedListToStringArray(
        StringUtils.trimAllWhitespace(properties.getBasename())));
    }
    if (properties.getEncoding() != null) {
        messageSource.setDefaultEncoding(properties.getEncoding().name());
    }
    messageSource.setFallbackToSystemLocale(properties.isFallbackToSystemLocale());
    Duration cacheDuration = properties.getCacheDuration();
    if (cacheDuration != null) {
        messageSource.setCacheMillis(cacheDuration.toMillis());
    }
    messageSource.setAlwaysUseMessageFormat(properties.isAlwaysUseMessageFormat());
    messageSource.setUseCodeAsDefaultMessage(properties.isUseCodeAsDefaultMessage());
    return messageSource;
}
```

我们只需要根据该类的properties配置类提供相应的语言配置文件的所在地：

```properties
spring.messages.basename=i18n.login
```

以上源码中可以看出：

- Spring Boot 将 MessageSourceProperties 以组件的形式添加到容器中；
- MessageSourceProperties 的属性与配置文件中以“spring.messages”开头的配置进行了绑定；
- ==Spring Boot 从容器中获取 MessageSourceProperties 组件，并从中读取国际化资源文件的  basename（文件基本名）、encoding（编码）等信息，将它们封装到 ResourceBundleMessageSource 中==；
- ==Spring Boot 将 ResourceBundleMessageSource 以组件的形式添加到容器中==，进而实现对国际化资源文件的管理。

##### （3）区域信息（国际化）解析器自动配置

我们知道，Spring MVC 进行国际化时有 2 个十分重要的对象：

- Locale：区域信息对象
- LocaleResolver：区域信息解析器，容器中的组件，负责获取区域信息对象


 我们可以通过以上两个对象对区域信息的切换，以达到切换语言的目的。

Spring Boot 在 WebMvcAutoConfiguration 中为区域信息解析器（LocaleResolver）进行了自动配置，源码如下：

```java
@Bean    
@ConditionalOnMissingBean(name = DispatcherServlet.LOCALE_RESOLVER_BEAN_NAME)   @SuppressWarnings("deprecation")    
public LocaleResolver localeResolver(){    
    // 如果用户自己配置了解析器就用用户的，没有就条件不成立，用mvc自己的   
    if(this.mvcProperties.getLocaleResolver() == WebMvcProperties.LocaleResolver.FIXED){   		  return new FixedLocaleResolver(this.mvcProperties.getLocale());
    }        
    //mvc自己的区域信息解析器
    AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();        	 Locale locale = (this.webProperties.getLocale() != null) ? this.webProperties.getLocale():this.mvcProperties.getLocale();        						localeResolver.setDefaultLocale(locale);
    return localeResolver;
}
```

我们看一下mvc的解析器AcceptHeaderLocaleResolver的规则：

```java
public Locale resolveLocale(HttpServletRequest request) {
    Locale defaultLocale = this.getDefaultLocale();
    // 通过请求头信息来判断区域语言
    if (defaultLocale != null && request.getHeader("Accept-Language") == null) {
        return defaultLocale;
    } else {
        Locale requestLocale = request.getLocale();
        List<Locale> supportedLocales = this.getSupportedLocales();
        if (!supportedLocales.isEmpty() && !supportedLocales.contains(requestLocale)) {
            Locale supportedLocale = this.findSupportedLocale(request, supportedLocales);
            if (supportedLocale != null) {
                return supportedLocale;
            } else {
                return defaultLocale != null ? defaultLocale : requestLocale;
            }
        } else {
            return requestLocale;
        }
    }
}
```

因而，如果我们想通过点击切换语言，官方提供的解析器是行不通的，可以自己写一个解析器：

1. 我们可以先创建按钮，通过点击按钮传递参数来切换语言：

```html
<!--thymeleaf 模板引擎的参数用（）代替 ？-->
<a class="btn btn-sm" th:href="@{/index.html(l='zh_CN')}">中文</a>
<a class="btn btn-sm" th:href="@{/index.html(l='en_US')}">English</a>
```

2. 编写解析器：

```java
//继承LocaleResolver就会被认为是一个区域信息解析器
    static class MyLocaleResolver implements LocaleResolver {
        @Override
        public Locale resolveLocale(HttpServletRequest request) {
            String lang = request.getParameter("l");
            Locale locale = Locale.getDefault();
            if(!StringUtils.isEmpty(lang)){
                // 获取语言与国家
                String[] locale1 = StringUtils.split(lang, "_");
                // 构造locale
                locale = new Locale(locale1[0],locale1[1]);
            }
            return locale;
        }

        @Override
        public void setLocale(HttpServletRequest request, HttpServletResponse response, Locale locale) {

        }
    }
```

3. 将解析器注入到容器中：

```java
@Bean
@ConditionalOnMissingBean(MyLocaleResolver.class)
public MyLocaleResolver myLocaleResolver(){
    return new MyLocaleResolver();
}
```

### 六、   配置Druid数据源

#### （1）引入依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.11</version>
</dependency>
```

#### （2）配置

##### ~1.对数据源进行配置

springboot默认数据源为hikari，想要更换数据源，只需要在spring.datasource.type指定即可。

在配置文件中，我们可以对druid的多个属性进行配置：

```yaml
spring:
  datasource:
#   数据源基本配置
    username: root
    password: root
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/duid
    type: com.alibaba.druid.pool.DruidDataSource
#   数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
#   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙  
    filters: stat,wall
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true  
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
```

==虽然我们配置了druid连接池的其它属性，但是不会生效。因为默认是使用的java.sql.Datasource的类来获取属性的，有些属性datasource没有。如果我们想让配置生效，需要手动创建Druid的配置文件，并将我们的配置注入其中，再将它加入到容器中==：

```java
@Bean
@ConfigurationProperties(prefix = "spring.datasource")
public DruidDataSource druidDataSource(){
    return new DruidDataSource();
}
```

##### ~2.  配置数据状态监控Servlet

```java
/**
 * 配置数据状态监控Servlet
 */
@Bean
public ServletRegistrationBean<StatViewServlet>statViewServlet(){
    // 设置Servlet和其url
    ServletRegistrationBean<StatViewServlet> bean = new ServletRegistrationBean<>(new StatViewServlet(),"/druid/*");
    // 设置的字段固定
    HashMap<String, String> initParameters = new HashMap<>();
    //设置允许访问的ip
    initParameters.put("allow","localhost");
    // 添加IP黑名单，当白名单和黑名单重复时，黑名单优先级更高  bean.addInitParameter("deny", "127.0.0.1");

    // 添加控制台管理用户
    bean.addInitParameter("loginUsername", "root");
    bean.addInitParameter("loginPassword", "123456");
    // 是否能够重置数据
    bean.addInitParameter("resetEnable", "false");
    bean.setInitParameters(initParameters);
    return bean;
}
```

SpringBoot取消了web.xml，我们想要注册一个Servlet，就可以使用ServletRegistrationBean注入到容器中实现。

当然，我们这里注入的是druid中数据监控站的servlet：StatViewServlet。

可能就会有人好奇，它的网页在哪呢，打开StatViewServlet，我们可以看到资源路径：

![image-20220712170540041](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220712170540041.png)

再看一下druid下的这个路径，就会发现网页资源就在这里：

![image-20220712170625783](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220712170625783.png)

#### （3）小结

当配置文件中配置了dataSource且不指定type时，springboot会先检查容器是否有用户自己注入的数据源，如果没有，则注入hikari作为数据源，如果有就不注入了。总而言之，只要用户自己注册了dataSource就用用户的，没有就用type指定的，type也不指定就用默认的。

### 七、SpringSecurity

#### 1.   概述

​		SpringSecurity是一个认证与授权的重量级安全框架，==其底层是一大堆过滤器（Filter）==，实现了用户认证与授权的一系列功能，并支持自定义服务。实际上该框架就是由==过滤器链==环环组成。

#### 2.   基本使用

我们可以配置一个SpringSecurity的基本配置类，只需要在我们的配置类上加@EnableWebSecurity，并且继承WebSecurityConfigurerAdapter即可开始配置。

##### （1）认证

​		重写方法：

```java
protected void configure(HttpSecurity http)
```

##### （2）授权

重写方法：

```java
protected void configure(AuthenticationManagerBuilder auth)
```

#### 3.   原理

SpringSecurity对于用户名密码的检查基于一个过滤器：==UsernamePasswordAuthenticationFilter==

他规定了用户提交表单的检查，用户名的name属性固定为：username；密码password属性固定为：password；方法固定为：post；url固定为：/login（也就是我们不需要自己写/login，只需提交到/login即可，当然也可以自己定义，这里不做拓展）。

对于怎么配置不做解释，自己去学。

### 八、shiro

#### 1.  概述

shiro是apache的一个轻量级安全框架，可以分为三大核心：Subject（用户主体），SecurityManager（安全管理器），Realm（操作数据的模块）。

#### 2.   基本配置

依赖：

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-web</artifactId>
    <version>1.9.0</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring</artifactId>
    <version>1.9.0</version>
</dependency>
```

##### （1）Realm制定认证与授权规则

自定义Realm继承AuthorizingRealm：

```java
public class MyRealm extends AuthorizingRealm {
    // 授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {

        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        // Subject代表当前用户
        Subject subject = SecurityUtils.getSubject();
        Object currentUser = subject.getPrincipal();
		// 给用户添加root权限，真实环境下权限应从数据库中取
        info.addStringPermission("root");
        return info;
    }
	//认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        String username = "root";
        String password = "123456";
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;

        // 用户名认证
        if(!token.getUsername().equals(username)){
            return null;   //return null会抛出UnknownAccountException异常 表示用户名不存在
        }
        // 验证密码的工作由shiro完成
        return new SimpleAuthenticationInfo("",password,"");
    }
}
```

##### （2）spring配置

```java
@Configuration
public class ShiroConfig {
	//对url访问权限由ShiroFilterFactoryBean完成，这个bean决定了对哪些url用哪些过滤器
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(@Qualifier("securityManager")SecurityManager securityManager){
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        bean.setSecurityManager(securityManager);
        bean.setLoginUrl("/toLogin");
        Map<String,String> map = new LinkedHashMap<>();
		// 不设过滤器 对所有用户开放
        map.put("/toLogin","anon");
        map.put("/doLogin","anon");

        // 顺序问题：当两个拦截路径出现重合 如这里的/**与/root，那么谁先写在前面就以谁的配置为准
        // 猜测shiro配置拦截规则时会先去判断路径现在有没有拦截规则 没有才去配
        //对/root访问做只有拥有“root”权限才能访问的过滤器
        map.put("/root","perms[root]");
        // 对所有url做登录才能访问的过滤器
        map.put("/**","authc");
        bean.setFilterChainDefinitionMap(map);
        return bean;
    }
	//注入安全管理器
    @Bean
    public SecurityManager securityManager(@Qualifier("myRealm") MyRealm myRealm){
        DefaultWebSecurityManager manager = new DefaultWebSecurityManager();
        manager.setRealm(myRealm);
        return manager;
    }
	//注入自定义Realm
    @Bean
    public MyRealm myRealm(){
        return new MyRealm();
    }
}
```

##### （3）访问控制

以登录验证为例：

```java
	@RequestMapping("/doLogin")
    public String login(String username, String password, Model model){
        // 获取当前访问的用户
        Subject subject = SecurityUtils.getSubject();
        //通过username与password生成token
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        try{
            //当前subject的login方法传入token实行登录（这里登录操作会分别进入Realm中的认证与授权方法判断）
            // 在Realm的protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken)方法中
            //看到UsernamePasswordToken是AuthenticationToken的一个实现类
            //因此传入的token可以在Realm的认证方法中使用判断
            subject.login(token);
            return "index";
        }catch (UnknownAccountException exception){
            model.addAttribute("msg","用户名错误");
            return "login";
        }catch (IncorrectCredentialsException exception){
            model.addAttribute("msg","密码错误");
            return "login";
        }
    }
```

顺带一提：==Realm认证方法中return null会抛出异常UnknownAccountException，而在最后的return SimpleAuthenticationInfo中，如果密码错误则会抛出IncorrectCredentialsException异常，因此可以根据两个异常直接判断用户名或密码错误==

### 九、Swagger

#### 1.   概述

​		在前后端分离时代，后端免不了设计一些API接口，为了解决前后端交流的效率问题，API文档是不可或缺的，因此，如果有一个框架能够帮助我们自动生成文档，那将事半功倍。

​		在开启API文档的话题前，需要介绍几个概念：

* **[OpenAPI](https://link.juejin.cn?target=https%3A%2F%2Fwww.openapis.org)** 是一个联合组织，负责制定标准的、开放的 API 描述规范。使用 openAPI 规范的 API 描述文件通常可以兼容各种相关软件或系统。

* **[Swagger](https://link.juejin.cn?target=https%3A%2F%2Fswagger.io%2F)** 一开始是一个组织，制定了最流行的 API 描述格式，并将其捐献给 OpenAPI 组织衍化成最初的 OpenAPI 标准。同时提供一些列的周边工具，包括著名的 `swagger-ui`。后来被 [SmartBear](https://link.juejin.cn?target=https%3A%2F%2Fsmartbear.com%2F) 公司收购，又提供了 `Swagger Hub` 等商业产品。

* **[Springfox](https://link.juejin.cn?target=http%3A%2F%2Fspringfox.github.io%2Fspringfox%2F)** 是一个适用于 Spring 的库，能够自动化生成 API 描述文件，支持 Swagger2, Swagger3 以及 OpenAPI3 三种格式。

* **[Springdoc](https://link.juejin.cn?target=https%3A%2F%2Fspringdoc.org%2F)** 是另一个适用于 Spring 的库，能生成 OpenAPI3 格式的 API 描述文件。

==总结一下就是 `OpenAPI` 是一个联盟与规范，`Springfox` 与 `Springdoc` 都是这个规范的实现。`Swagger` 是一个公司，贡献了许多好用的工具并且参与制定了 OpenAPI 规范。==

Swagger是一个API接口文档框架，支持自动动态地生成API接口文档，这里是Swagger2的依赖：

```xml
<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger-ui</artifactId>
	<version>2.9.2</version>
</dependency>

<dependency>
	<groupId>io.springfox</groupId>
	<artifactId>springfox-swagger2</artifactId>
	<version>2.9.2</version>
</dependency>
```

需要注意的是：

​		==在SpringBoot2.6之后，Spring MVC 处理程序映射匹配请求路径的默认策略已从 AntPathMatcher 更改为PathPatternParser。如果需要切换为AntPathMatcher，官方给出的方法是配置spring.mvc.pathmatch.matching-strategy=ant_path_matcher==

​		但是actuator endpoints在2.6之后也使用基于 PathPattern 的 URL 匹配，而且actuator endpoints的路径匹配策略无法通过配置属性进行配置，如果同时使用Actuator和Springfox，会导致程序启动失败，所以只是进行上面的设置是不行的。

​		简单来说：就是springboot2.6之后上面的依赖在boot启动时会出现异常。

​		下面介绍解决方法。

#### 2.   springdoc

​		对于springboot高版本swagger2异常问题，这里推荐一个我认为最好的解决方案：==换框架==

​		由于网上有许多Swagger2的教程，这里不介绍Swagger2，而是介绍一下springdoc（基于Swagger3以及拥有OpenApi3规范的Api文档框架）。

​		springdoc是一个十分优秀的API文档框架（swagger的拓展，本质还是以swagger为主），集成了swagger，并且能生成符合OpenAPI3规范的API描述文件，在此介绍一下：

​		在springboot项目中集成，依赖：

```xml
<!--springdoc-->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-ui</artifactId>
    <version>1.6.8</version>
</dependency>
```

​		可以配置swagger-ui的路径：

```yaml
springdoc:
  swagger-ui:
    path: swagger
```

坑点（翻源码找了10分钟才发现错误）：也许是版本不同，刚开始看文档写的是全路径：http//localhost:8080/swagger，但这样会出错，来看一下源码：

在springdoc-openapi-ui包下有SwaggerUiHome类，以下规定了swagger-ui的路径：

![image-20220724162616864](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220724162616864.png)

因此，如果说你想http://localhost:8080/swagger直接访问swagger-ui，在yml中直接写swagger就好。

#### 3.  配置

​		如果项目只有一个开发者，那么一个说明文档就行；如果是协同开发，就需要各写各的文档。

​		需要把OpenAPI类（单人开发）或GroupedOpenApi类（协同开发）注入到spring容器：

​		以下为Config类：

```java
@Configuration
public class SwaggerConfig {
    @Bean
    public GroupedOpenApi springOpenAPI(){
        // 多人协同开发环境下 一个GroupedOpenApi代表一个人或一个开发组的Api文档
        // 其中规定了对哪些包、哪些路径等的Api进行扫描制作文档
        return GroupedOpenApi.builder()
            	//你的组名
                .group("hoshino")
                .pathsToMatch("/**")
                .packagesToScan("com.hoshino")
                .build();
        
        //单人开发环境下
//        return new OpenAPI()
//                .info(new Info().title("SpringShop API")
//                        .description("Spring shop sample application")
//                        .version("v0.0.1")
//                        .license(new License().name("Apache 2.0")
//                        .url("http://springdoc.org")))
//                .externalDocs(new ExternalDocumentation()
//                        .description("SpringShop Wiki Documentation")
//                        .url("https://springshop.wiki.github.org/docs"));
    }
}
```

另外：如果是OpenAPI，也可以不需要自己注入Bean，可以在配置文件中进行简单规定。

提案：API文档一般在开发环境下使用，在生产环境下禁用，在springdoc下，可以在配置文件中禁用：

```yaml
springdoc:
	api-docs: 
		enabled: false
# 或
springdoc:
	swagger-ui:
		enabled: false
```



#### 4.   swagger2与springdoc（openApi3）对应注解

| swagger2           | OpenAPI 3                                                    | 注解位置                         |
| ------------------ | ------------------------------------------------------------ | -------------------------------- |
| @Api               | @Tag(name = “接口类描述”)                                    | Controller 类上                  |
| @ApiOperation      | @Operation(summary =“接口方法描述”)                          | Controller 方法上                |
| @ApiImplicitParams | @Parameters                                                  | Controller 方法上                |
| @ApiImplicitParam  | @Parameter(description=“参数描述”)                           | Controller 方法上 @Parameters 里 |
| @ApiParam          | @Parameter(description=“参数描述”)                           | Controller 方法的参数上          |
| @ApiIgnore         | @Parameter(hidden = true) 或 @Operation(hidden = true) 或 @Hidden | -                                |
| @ApiModel          | @Schema                                                      | DTO类上                          |
| @ApiModelProperty  | @Schema                                                      | DTO属性上                        |

### 十、分布式系统

#### 1.  概述

​		当单机环境已经难以满足并发量需求时，可以基于网络将多个单机组成分布式系统，通过负载均衡、远程调用等方式提高系统承载量。分布式系统理论最早由Google提出。

#### 2.  RPC

​		RPC是通过网络通信的方式执行远程调用的一种协议。底层基于原生tcp协议。相比于http通信，rpc有如下特点：

1. 使用rpc框架实现服务间调用时，要求提供方与消费方都必须使用统一的rpc框架。
2. 调用快，处理快。

![preview](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/v2-52a0106c08263acecb3dedd36fbe0dd2_r.jpg)

#### 3.  dubbo3

##### （1）基本介绍

​		dubbo是一个rpc框架，基本执行流程如下：

![arch-service-discovery](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/architecture.png)

​		可以看出，除了远程调用的时候是同步执行，其余操作都是异步完成，其中有个关键点是==Registry（注册中心）==，从图中可以看出，框架的第一步就是服务提供者将服务注册到注册中心，然后消费者远程调用服务都是去请求注册中心来完成，一般来讲，注册中心可以选==zookeeper（分布式系统的协调管理框架）==。

##### （2）环境依赖

引入依赖：

```xml
<properties>
    <dubbo.version>3.0.8</dubbo.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
        <version>${dubbo.version}</version>
    </dependency>
    <!-- This dependency helps to introduce Curator and Zookeeper dependencies that are necessary for Dubbo to work with zookeeper as transitive dependencies  -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <version>${dubbo.version}</version>
        <type>pom</type>
    </dependency>
</dependencies>
```

从官方提示中可以得到：dubbo-dependencies-zookeeper依赖是引入dubbo需要的zookeeper的客户端依赖，减少用户使用dubbo的成本，如果遇到版本兼容问题，用户也可以不使用这个依赖，而是自行引入Curator、Zookeeper Client 等依赖。

如果要开发springboot应用，还需引入启动器：

```xml
<!-- dubbo starter -->
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
</dependency>
```

##### （3）基本使用

* 提供者端：

1. 基本配置文件：

```yaml
dubbo:
  application:
    name: dubbo-springboot-demo-provider
  protocol:
    name: dubbo
    port: -1
  registry:
    id: zk-registry
    address: zookeeper://127.0.0.1:2181
  config-center:
    address: zookeeper://127.0.0.1:2181
  metadata-report:
    address: zookeeper://127.0.0.1:2181
```

2. 在想要提供的服务上加上@DubboService：

```java
@DubboService
public class DemoServiceImpl implements DemoService {
    @Override
    public String sayHello(String name) {
        System.out.println("Hello " + name + ", request from consumer: " + RpcContext.getContext().getRemoteAddress());
        return "Hello " + name;
    }
}
```

3. 在启动类上加上@EnableDubbo

```java
@SpringBootApplication
@EnableDubbo
public class ProviderApplication {
    public static void main(String[] args) throws Exception {
        new EmbeddedZooKeeper(2181, false).start();

        SpringApplication.run(ProviderApplication.class, args);
        System.out.println("dubbo service started");
        new CountDownLatch(1).await();
    }
}
```

* 消费者端：

1. 基本配置文件：

```yaml
dubbo:
  application:
    name: dubbo-springboot-demo-consumer
  protocol:
    name: dubbo
    port: -1
  registry:
    id: zk-registry
    address: zookeeper://127.0.0.1:2181
  config-center:
    address: zookeeper://127.0.0.1:2181
  metadata-report:
    address: zookeeper://127.0.0.1:2181
```

2. 在启动类上加上@EnableDubbo；在希望远程调用的类上加上@DubboReference（==注意，远程调用的类应与提供者端的服务接口的类名称相同，路径相同==）：

```java
@SpringBootApplication
@Service
@EnableDubbo
public class ConsumerApplication {

    @DubboReference
    private DemoService demoService;

    public static void main(String[] args) {

        ConfigurableApplicationContext context = SpringApplication.run(ConsumerApplication.class, args);
        ConsumerApplication application = context.getBean(ConsumerApplication.class);
        String result = application.doSayHello("world");
        System.out.println("result: " + result);
    }

    public String doSayHello(String name) {
        return demoService.sayHello(name);
    }
}
```

### 十一、日志与日志门面

#### 1.  基本介绍

##### （1）问题引入

​		日志大家都懂，但在微服务架构上，可能我的A微服务使用的是logback日志；B微服务使用的是log4j；C微服务使用的又是log4j2。这样，我们的系统就不得不维护三种日志框架，非常麻烦。于是日志门面是一种好的解决方式。

##### （2）日志门面

​		日志门面的解决方案采用门面模式（也就是外观模式）：

![img](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/801753-20180321204740208-1670144043.png)

​		slf4j就是java原生的一个日志门面。

​		首先说明：==slf4j只是一个日志标准，并不是日志系统的具体实现==。它只是提供了一组接口，让具体的日志实现类去实现这组接口，然后由LoggerFactory去决定使用哪个日志实现类。用户只需要通过LoggerFactory获得Logger就可以使用了。

#### 2.   日志门面细节说明

​		当我们的系统中没有任何日志实现类时，想都不用想，这时调用日志肯定会报错，但需要说明一点：springboot自带logback日志实现。

​		但只要引入一个日志，此时slf4j下的LoggerFactory便能找到并使用，需要说明的是：==即使引入多个日志实现也不会报错，slf4j只是会在程序运行开始时警告说系统有多个日志实现，随后在其中选择一个使用。==

#### 3.  slf4j实现原理

​		slf4j用法就是万年不变的一句：**Logger logger = LoggerFactory.getLogger(xxx.class);**

​		我们跟进一下源码：

​		在LoggerFactory下有一个bind方法，其中一句为关键：

```java
//找出所有日志实现的路径（由于日志实现可能不止一个，所以返回Set）
staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
```

​		进到这个方法中看一下：

```java
static Set<URL> findPossibleStaticLoggerBinderPathSet() {
     // use Set instead of list in order to deal with bug #138
     // LinkedHashSet appropriate here because it preserves insertion order
     // during iteration
     Set<URL> staticLoggerBinderPathSet = new LinkedHashSet<URL>();
     try {
         ClassLoader loggerFactoryClassLoader = LoggerFactory.class.getClassLoader();
         Enumeration<URL> paths;
         if (loggerFactoryClassLoader == null) {
             paths = ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH);
         } else {
             //关键句
             paths = loggerFactoryClassLoader.getResources(STATIC_LOGGER_BINDER_PATH);
         }
         while (paths.hasMoreElements()) {
             URL path = paths.nextElement();
			 staticLoggerBinderPathSet.add(path);
         }
     } catch (IOException ioe) {
         Util.report("Error getting resources from path", ioe);
     }
     return staticLoggerBinderPathSet;
 }
```

在这个方法的关键句的STATIC_LOGGER_BINDER_PATH的值为==“org/slf4j/impl/StaticLoggerBinder.class”==，

因此我们可以得到：==所有的slf4j的实现，在提供的jar包路径下，一定有org/slf4j/impl/StaticLoggerBinder.class存在==：

![image-20220727143700463](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220727143700463.png)

![image-20220727143819034](https://raw.githubusercontent.com/hoshinojyunn/PicBed/main/image-20220727143819034.png)
