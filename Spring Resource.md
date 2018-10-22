# Spring Resource    

### 前言  

​	在日常程序开发中，处理外部资源是很繁琐的事情，我们可能需要处理URL资源、File资源资源、`ClassPath`相关资源、服务器相关资源（JBoss AS 5.x上的VFS资源）等等很多资源。因此处理这些资源需要使用不同的接口，这就增加了我们系统的复杂性；而且处理这些资源步骤都是类似的（打开资源、读取资源、关闭资源），因此如果能抽象出一个统一的接口来对这些底层资源进行统一访问，是不是很方便，而且使我们系统更加简洁，都是对不同的底层资源使用同一个接口进行访问。

​       Spring 提供一个Resource接口来统一这些底层资源一致的访问，而且提供了一些便利的接口，从而能提供我们的生产力。

### Resource 接口

- Spring的Resource接口代表底层外部资源，提供了对底层外部资源的一致性访问接口。

  ```java
  public interface InputStreamSource {  
      InputStream getInputStream() throws IOException;  
  } 
  ```

- InputStreamSource接口解析

  - **getInputStream**：每次调用都将返回一个新鲜的资源对应的`java.io. InputStream`字节流，调用者在使用完毕后必须关闭该资源。
  - Resource接口继承`InputStreamSource`接口，并提供一些便利方法：
    - **exists**：返回当前Resource代表的底层资源是否存在，true表示存在。
    - **isReadable**：返回当前Resource代表的底层资源是否可读，true表示可读。
    - **isOpen**：返回当前Resource代表的底层资源是否已经打开，如果返回true，则只能被读取一次然后关闭以避免资源泄露；常见的Resource实现一般返回false。
    - **getURL**：如果当前Resource代表的底层资源能由java.util.URL代表，则返回该URL，否则抛出IOException。
    - **getURI**：如果当前Resource代表的底层资源能由java.util.URI代表，则返回该URI，否则抛出IOException。
    - **getFile**：如果当前Resource代表的底层资源能由java.io.File代表，则返回该File，否则抛出IOException。
    - **contentLength**：返回当前Resource代表的底层文件资源的长度，一般是值代表的文件资源的长度。
    - **lastModified**：返回当前Resource代表的底层资源的最后修改时间。
    - **createRelative**：用于创建相对于当前Resource代表的底层资源的资源，比如当前Resource代表文件资源“d:/test/”则createRelative（“test.txt”）将返回表文件资源“d:/test/test.txt”Resource资源。
    - **getFilename**：返回当前Resource代表的底层文件资源的文件路径，比如File资源“file://d:/test.txt”将返回“d:/test.txt”，而URL资源http://www.javass.cn将返回“”，因为只返回文件路径。
    - **getDescription**：返回当前Resource代表的底层资源的描述符，通常就是资源的全路径（实际文件名或实际URL地址）。    
  - Resource接口提供了足够的抽象，足够满足我们日常使用。而且提供了很多内置Resource实现：`ByteArrayResource、InputStreamResource 、FileSystemResource 、UrlResource 、ClassPathResource、ServletContextResource、VfsResource`等。

  ```java
  public interface Resource extends InputStreamSource {  
         boolean exists();  
         boolean isReadable();  
         boolean isOpen();  
         URL getURL() throws IOException;  
         URI getURI() throws IOException;  
         File getFile() throws IOException;  
         long contentLength() throws IOException;  
         long lastModified() throws IOException;  
         Resource createRelative(String relativePath) throws IOException;  
         String getFilename();  
         String getDescription();  
  }  
  ```

- Resource 和 @Autowired的不同

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

