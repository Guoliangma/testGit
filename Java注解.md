# Java 注解    

### 前言  

​	注解是Java 5的一个新特性，注解(`Annotation`)是一种应用于类、方法、参数、变量、构造器及包声明中的特殊修饰符，它是一种由JSR-175标准选择用来描述元数据的一种工具。在注解出现之前，程序的元数据只是通过java注释和javadoc，但是注解提供的功能要远远超过这些。注解不仅包含了元数据，它还可以作用于程序运行过程中、注解解释器可以通过注解决定程序的执行顺序

### 注解  

- 什么是注解

  注解是插入你代码中的一种注释或者说是一种元数据（meta data）。这些注解信息可以在编译期使用预编译工具进行处理（pre-compiler tools），也可以在运行期使用Java反射机制进行处理。下面是一个类注解的例子：

  ````java
  @MyAnnotation(name="someName",  value = "Hello World")
  	public class TheClass {
  }
  ````

  在`TheClass`类定义的上面有一个`@MyAnnotation`的注解。注解的定义与接口的定义相似，下面是`MyAnnotation`注解的定义：

  ````java
  @Retention(RetentionPolicy.RUNTIME)
  @Target(ElementType.TYPE)
  public @interface MyAnnotation {
    public String name();
    public String value();
  }
  ````

  在interface前面的@符号表名这是一个注解，一旦你定义了一个注解之后你就可以将其应用到你的代码中，就像之前我们的那个例子那样。
  在注解定义中的两个指示`@Retention(RetentionPolicy.RUNTIME)`和`@Target(ElementType.TYPE)`，说明了这个注解该如何使用。
  `@Retention(RetentionPolicy.RUNTIME)`表示这个注解可以在运行期通过反射访问。如果你没有在注解定义的时候使用这个指示那么这个注解的信息不会保留到运行期，这样反射就无法获取它的信息。
  `@Target(ElementType.TYPE)` 表示这个注解只能用在类型上面（比如类跟接口）。你同样可以把Type改为`Field`或者`Method`，或者你可以不用这个指示，这样的话你的注解在类，方法和变量上就都可以使用了。

- 类注解  

  你可以在运行期访问类，方法或者变量的注解信息，下是一个访问类注解的例子：

  ````java
  Class aClass = TheClass.class;
  Annotation[] annotations = aClass.getAnnotations();
  for(Annotation annotation : annotations){
      if(annotation instanceof MyAnnotation){
          MyAnnotation myAnnotation = (MyAnnotation) annotation;
          System.out.println("name: " + myAnnotation.name());
          System.out.println("value: " + myAnnotation.value());
      }
  }
  ````

  你还可以像下面这样指定访问一个类的注解：

  ````java
  Class aClass = TheClass.class;
  Annotation annotation = aClass.getAnnotation(MyAnnotation.class);
  if(annotation instanceof MyAnnotation){
      MyAnnotation myAnnotation = (MyAnnotation) annotation;
      System.out.println("name: " + myAnnotation.name());
      System.out.println("value: " + myAnnotation.value());
  }

  ````

- 方法注解

  下面是一个方法注解的例子：

  ````java
  public class TheClass {
    @MyAnnotation(name="someName",  value = "Hello World")
    public void doSomething(){}
  }
  ````

  你可以像这样访问方法注解：

  ````java
  Method method = ... //获取方法对象
  Annotation[] annotations = method.getDeclaredAnnotations();
  for(Annotation annotation : annotations){
      if(annotation instanceof MyAnnotation){
          MyAnnotation myAnnotation = (MyAnnotation) annotation;
          System.out.println("name: " + myAnnotation.name());
          System.out.println("value: " + myAnnotation.value());
      }
  }
  ````

  你可以像这样访问指定的方法注解：

  ````java
  Method method = ... // 获取方法对象
  Annotation annotation = method.getAnnotation(MyAnnotation.class);
  if(annotation instanceof MyAnnotation){

      MyAnnotation myAnnotation = (MyAnnotation) annotation;
      System.out.println("name: " + myAnnotation.name());
      System.out.println("value: " + myAnnotation.value());
  }

  ````

- 参数注解

  方法参数也可以添加注解，就像下面这样：

  ````java
  public class TheClass {
    public static void doSomethingElse(
          @MyAnnotation(name="aName", value="aValue") String parameter){
    }
  }
  ````

  你可以通过Method对象来访问方法参数注解：

  ````java
  Method method = ... //获取方法对象
  Annotation[][] parameterAnnotations = method.getParameterAnnotations();
  Class[] parameterTypes = method.getParameterTypes();
  int i=0;
  for(Annotation[] annotations : parameterAnnotations){
    Class parameterType = parameterTypes[i++];
    for(Annotation annotation : annotations){
      if(annotation instanceof MyAnnotation){
          MyAnnotation myAnnotation = (MyAnnotation) annotation;
          System.out.println("param: " + parameterType.getName());
          System.out.println("name : " + myAnnotation.name());
          System.out.println("value: " + myAnnotation.value());
      }
    }
  }
  ````

  需要注意的是`Method.getParameterAnnotations()`方法返回一个注解类型的二维数组，每一个方法的参数包含一个注解数组。

- 变量注解

  下面是一个变量注解的例子：

  ````java
  public class TheClass {
    @MyAnnotation(name="someName",  value = "Hello World")
    public String myField = null;
  }
  ````

  你可以像这样来访问变量的注解：

  ````java
  Field field = ... //获取方法对象
  Annotation[] annotations = field.getDeclaredAnnotations();
  for(Annotation annotation : annotations){
   if(annotation instanceof MyAnnotation){
   MyAnnotation myAnnotation = (MyAnnotation) annotation;
   System.out.println("name: " + myAnnotation.name());
   System.out.println("value: " + myAnnotation.value());
   }
  }
  ````

  你可以像这样访问指定的变量注解：

  ````java
  Field field = ...//获取方法对象
  Annotation annotation = field.getAnnotation(MyAnnotation.class);
  if(annotation instanceof MyAnnotation){
   MyAnnotation myAnnotation = (MyAnnotation) annotation;
   System.out.println("name: " + myAnnotation.name());
   System.out.println("value: " + myAnnotation.value());
  }
  ````

- Resource 和 @Autowired 的不同

  - @Autowired与@Resource都可以用来装配bean. 都可以写在字段上,或写在setter方法上。

  - @Autowired默认按类型装配（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如：
    @Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用，如下：

    ```java
    @Autowired() @Qualifier("baseDao")    
    private BaseDao baseDao;  
    ```

  - @Resource 是JDK1.6支持的注解**，**默认按照名称进行装配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。只不过注解处理器我们使用的是Spring提供的，是一样的，无所谓解耦不解耦的说法，两个在便利程度上是等同的。

  - 他们的主要区别就是@Autowired是默认按照类型装配的 @Resource默认是按照名称装配的
    byName 通过参数名 自动装配，如果一个bean的name 和另外一个bean的 property 相同，就自动装配。
    byType 通过参数的数据类型自动自动装配，如果一个bean的数据类型和另外一个bean的property属性的数据类型兼容，就自动装配

    ```java
    @Resource(name="baseDao")    
    private BaseDao baseDao; 
    ```

  - 我们可以通过 @Autowired 或 @Resource 在 Bean 类中使用自动注入功能，但是 Bean 还是在 XML 文件中通过 <bean> 进行定义 —— 也就是说，在 XML 配置文件中定义 Bean，通过@Autowired 或 @Resource 为 Bean 的成员变量、方法入参或构造函数入参提供自动注入的功能。
    比如下面的beans.xml。

    ```java
    public class Boss {
      private Car car;
      private Office office;

      // 省略 get/setter

      @Override
      public String toString() {
        return "car:" + car + "\n" + "office:" + office;
      }
    }
    ```

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:context="http://www.springframework.org/schema/context"
           xsi:schemaLocation="http://www.springframework.org/schema/beans 
                               http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
                               http://www.springframework.org/schema/context 
                               http://www.springframework.org/schema/context/spring-context-2.5.xsd">

      <context:annotation-config/> 

      <bean id="boss" class="com.wuxinliulei.Boss"/>
      <bean id="office" class="com.wuxinliulei.Office">
        <property name="officeNo" value="001"/>
      </bean>
      <bean id="car" class="com.wuxinliulei.Car" scope="singleton">
        <property name="brand" value=" 红旗 CA72"/>
        <property name="price" value="2000"/>
      </bean>
    </beans>
    ```

  ​

- 定义了三个bean对象，但是没有了我们书序的ref指向的内容
  比如

  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <beans xmlns="http://www.springframework.org/schema/beans"
  	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	xsi:schemaLocation="http://www.springframework.org/schema/beans 
   http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
      <bean id="boss" class="com.wuxinliulei.Boss">
          <property name="car" ref="car"/>
          <property name="office" ref="office" />
      </bean>
      <bean id="office" class="com.wuxinliulei.Office">
          <property name="officeNo" value="002"/>
      </bean>
      <bean id="car" class="com.wuxinliulei.Car" scope="singleton">
          <property name="brand" value=" 红旗 CA72"/>
          <property name="price" value="2000"/>
      </bean>
  </beans>
  ```

  ​

- spring2.5提供了基于注解（Annotation-based）的配置，我们可以通过注解的方式来完成注入依赖。在Java代码中可以使用 @Resource或者@Autowired注解方式来经行注入。虽然@Resource和@Autowired都可以来完成注入依赖，但它们之间是有区 别的。首先来看一下：

  - @Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入
  - @Autowired默认是按照类型装配注入的，如果想按照名称来转配注入，则需要结合@Qualifier一起使用
  - @Resource注解是由JDK提供，而@Autowired是由Spring提供
  - @Resource和@Autowired都可以书写标注在字段或者该字段的setter方法之上。  

- 使用注解的方式，我们需要修改spring配置文件的头信息如下:

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="Index of /schema/context"
         xsi:schemaLocation="Index of /schema/beans 
  http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
  Index of /schema/context
  http://www.springframework.org/schema/context/spring-context-2.5.xsd">
                 
  <context:annotation-config/>
  ```

  ​