# Java 反射    

### 前言  

​	Java反射机制可以让我们在编译期(Compile Time)之外的运行期(Runtime)检查类，接口，变量以及方法的信息。反射还可以让我们在运行期实例化对象，调用方法，通过调用get/set方法获取变量的值。

​	Java反射机制功能强大而且非常实用。举个例子，你可以用反射机制把Java对象映射到数据库表，或者把脚本中的一段语句在运行期映射到相应的对象调用方法上。

### Classes

- Class 对象

  使用Java反射机制可以在运行时期检查Java类的信息，检查Java类的信息往往是你在使用Java反射机制的时候所做的第一件事情，通过获取类的信息你可以获取以下相关的内容：

  - Class对象

  - 类名

  - 修饰符

  - 包信息

  - 父类

  - 实现的接口

  - 构造器

  - 方法

  - 变量

  - 注解

    Java反射的例子

    下面是一个Java反射的简单例子：

    ```java
    Method[] methods = MyObject.class.getMethods();
    for(Method method : methods){
      System.out.println("method = " + method.getName());
    }
    ```

    在这个例子中通过调用MyObject类的class属性获取对应的Class类的对象，通过这个Class类的对象获取MyObject类中的方法集合。迭代这个方法的集合并且打印每个方法的名字。

  在你想检查一个类的信息之前，你首先需要获取类的Class对象。Java中的所有类型包括基本类型(int, long, float等等)，即使是数组都有与之关联的Class类的对象。如果你在编译期知道一个类的名字的话，那么你可以使用如下的方式获取一个类的Class对象。

  ````java
  Class myObjectClass = MyObject.class;
  ````

  如果你在编译期不知道类的名字，但是你可以在运行期获得到类名的字符串,那么你则可以这么做来获取Class对象:

  ````java
  String className = ... ;//在运行期获取的类名字符串
  Class class = Class.forName(className);
  ````

  在使用Class.forName()方法时，你必须提供一个类的全名，这个全名包括类所在的包的名字。例如MyObject类位于com.jenkov.myapp包，那么他的全名就是com.jenkov.myapp.MyObject。
  如果在调用Class.forName()方法时，没有在编译路径下(classpath)找到对应的类，那么将会抛出ClassNotFoundException。

- 类名

  - 通过getName() 方法返回类的全限定类名（包含包名）：

    ````java
    Class aClass = ... //获取Class对象
    String className = aClass.getName();
    ````

  - 如果你仅仅只是想获取类的名字(不包含包名)，那么你可以使用getSimpleName()方法:

    ````java
    Class aClass = ... //获取Class对象，具体方式可见Class对象小节
    String simpleClassName = aClass.getSimpleName();
    ````

- 修饰符

  - 可以通过Class对象来访问一个类的修饰符，即public,private,static等等的关键字，你可以使用如下方法来获取类的修饰符：

    ```java
    Class  aClass = ... //获取Class对象，
    int modifiers = aClass.getModifiers();
    ```

  - 修饰符都被包装成一个int类型的数字，这样每个修饰符都是一个位标识(flag bit)，这个位标识可以设置和清除修饰符的类型。
    可以使用`java.lang.reflect.Modifier`类中的方法来检查修饰符的类型：

    ````java
    Modifier.isAbstract(int modifiers);
    Modifier.isFinal(int modifiers);
    Modifier.isInterface(int modifiers);
    Modifier.isNative(int modifiers);
    Modifier.isPrivate(int modifiers);
    Modifier.isProtected(int modifiers);
    Modifier.isPublic(int modifiers);
    Modifier.isStatic(int modifiers);
    Modifier.isStrict(int modifiers);
    Modifier.isSynchronized(int modifiers);
    Modifier.isTransient(int modifiers);
    Modifier.isVolatile(int modifiers);
    ````

- 包信息

  - 可以使用Class对象通过如下的方式获取包信息：

  ```java
  Class  aClass = ... //获取Class对象
  Package package = aClass.getPackage();
  ```

  ​

- 父类

  - 通过Class对象你可以访问类的父类，如下例：

    ````java
    Class superclass = aClass.getSuperclass();
    ````

    可以看到superclass对象其实就是一个Class类的实例，所以你可以继续在这个对象上进行反射操作。

- 实现的接口

  - 可以通过如下方式获取指定类所实现的接口集合：

  ```java
  Constructor[] constructors = aClass.getConstructors();
  ```

- 方法

  ````java
  Field[] method = aClass.getFields();
  ````

- 注解

  `````java
  Annotation[] annotations = aClass.getAnnotations();
  `````


###  构造器  

​	利用Java的反射机制你可以检查一个类的构造方法，并且可以在运行期创建一个对象。这些功能都是通过`java.lang.reflect.Constructor`这个类实现的。本节将深入的阐述`Java Constructor`对象。

- 获取Constructor对象

  我们可以通过Class对象来获取Constructor类的实例：

  ````java
  Class aClass = ...//获取Class对象
  Constructor[] constructors = aClass.getConstructors();
  ````

  返回的Constructor数组包含每一个声明为公有的（Public）构造方法。

  如果你知道你要访问的构造方法的方法参数类型，你可以用下面的方法获取指定的构造方法，这例子返回的构造方法的方法参数为String类型：

  ````java
  Class aClass = ...//获取Class对象
  Constructor constructor = aClass.getConstructor(new Class[]{String.class});
  ````

  如果没有指定的构造方法能满足匹配的方法参数则会抛出：`NoSuchMethodException`。

- 构造方法参数

  你可以通过如下方式获取指定构造方法的方法参数信息：

  ````java
  Constructor constructor = ... //获取Constructor对象
  Class[] parameterTypes = constructor.getParameterTypes();
  ````

- 利用Constructor对象实例化一个类

  你可以通过如下方法实例化一个类：

  ````java
  Constructor constructor = MyObject.class.getConstructor(String.class);
  MyObject myObject = (MyObject) constructor.newInstance("constructor-arg1");
  ````

  `constructor.newInstance()`方法的方法参数是一个可变参数列表，但是当你调用构造方法的时候你必须提供精确的参数，即形参与实参必须一一对应。在这个例子中构造方法需要一个String类型的参数，那我们在调用`newInstance`方法的时候就必须传入一个String类型的参数。

### 变量

​	使用Java反射机制你可以运行期检查一个类的变量信息(成员变量)或者获取或者设置变量的值。通过使用`java.lang.reflect.Field`类就可以实现上述功能。在本节会带你深入了解Field对象的信息。

- 获取Field对象

  可以通过`Class`对象获取`Field`对象，如下例：

  ````java
  Class aClass = ...//获取Class对象
  Field[] methods = aClass.getFields();
  ````

  返回的Field对象数组包含了指定类中声明为公有的(public)的所有变量集合。
  如果你知道你要访问的变量名称，你可以通过如下的方式获取指定的变量：

  ````java
  Class  aClass = MyObject.class
  Field field = aClass.getField("someField");
  ````

  上面的例子返回的Field类的实例对应的就是在`MyObject`类中声明的名为`someField`的成员变量，就是这样：

  ````java
  public class MyObject{
    public String someField = null;
  }
  ````

  在调用`getField()`方法时，如果根据给定的方法参数没有找到对应的变量，那么就会抛出`NoSuchFieldException`。

- 变量名称

  一旦你获取了Field实例，你可以通过调用`Field.getName()`方法获取他的变量名称，如下例：

  ````java
  Field field = ... //获取Field对象
  String fieldName = field.getName();
  ````

- 变量类型

  你可以通过调用Field.getType()方法来获取一个变量的类型（如String, int等等）

  ````java
  Field field = aClass.getField("someField");
  Object fieldType = field.getType();
  ````

- 获取或设置（get/set）变量值

  一旦你获得了一个Field的引用，你就可以通过调用Field.get()或Field.set()方法，获取或者设置变量的值，如下例：

  ````java
  Class  aClass = MyObject.class
  Field field = aClass.getField("someField");
  MyObject objectInstance = new MyObject();
  Object value = field.get(objectInstance);
  field.set(objetInstance, value);
  ````

  传入`Field.get()/Field.set()`方法的参数`objetInstance`应该是拥有指定变量的类的实例。在上述的例子中传入的参数是`MyObject`类的实例，是因为`someField`是`MyObject`类的实例。
  如果变量是静态变量的话`(public static)`那么在调用`Field.get()/Field.set()`方法的时候传入`null`做为参数而不用传递拥有该变量的类的实例。(译者注：你如果传入拥有该变量的类的实例也可以得到相同的结果)

### 方法  

​	使用Java反射你可以在运行期检查一个方法的信息以及在运行期调用这个方法，通过使用`java.lang.reflect.Method`类就可以实现上述功能。在本节会带你深入了解`Method`对象的信息。

- 获取Method对象

  以通过Class对象获取Method对象，如下例：

  ````java
  Class aClass = ...//获取Class对象
  Method[] methods = aClass.getMethods();
  ````

  返回的Method对象数组包含了指定类中声明为公有的(public)的所有变量集合。
  如果你知道你要调用方法的具体参数类型，你就可以直接通过参数类型来获取指定的方法，下面这个例子中返回方法对象名称是`doSomething`，他的方法参数是String类型：

  ````java 
  Class  aClass = ...//获取Class
  Method method = aClass.getMethod("doSomething", new Class[]{String.class});
  ````

  如果根据给定的方法名称以及参数类型无法匹配到相应的方法，则会抛出`NoSuchMethodException`。
  如果你想要获取的方法没有参数，那么在调用`getMethod()`方法时第二个参数传入null即可，就像这样：

  ````java
  Class  aClass = ...//获取Class对象
  Method method = aClass.getMethod("doSomething", null);
  ````

- 方法参数以及返回类型

  你可以获取指定方法的方法参数是哪些：

  ````java
  Method method = ... //获取Class对象
  Class[] parameterTypes = method.getParameterTypes();
  ````

  你可以获取指定方法的返回类型：

  ````java
  Method method = ... //获取Class对象
  Class returnType = method.getReturnType();
  ````

- 通过Method对象调用方法

  你可以通过如下方式来调用一个方法：

  ````java
  //获取一个方法名为doSomesthing，参数类型为String的方法
  Method method = MyObject.class.getMethod("doSomething", String.class);
  Object returnValue = method.invoke(null, "parameter-value1");

  ````

  传入的null参数是你要调用方法的对象，如果是一个静态方法调用的话则可以用null代替指定对象作为invoke()的参数，在上面这个例子中，如果`doSomething`不是静态方法的话，你就要传入有效的`MyObject`实例而不是`null`。
  `Method.invoke(Object target, Object … parameters)`方法的第二个参数是一个可变参数列表，但是你必须要传入与你要调用方法的形参一一对应的实参。就像上个例子那样，方法需要String类型的参数，那我们必须要传入一个字符串。

### Getters and Setters  

​	使用Java反射你可以在运行期检查一个方法的信息以及在运行期调用这个方法，使用这个功能同样可以获取指定类的getters和setters，你不能直接寻找getters和setters，你需要检查一个类所有的方法来判断哪个方法是getters和setters。

​	首先让我们来规定一下getters和setters的特性：

**Getter**

Getter方法的名字以get开头，没有方法参数，返回一个值。

**Setter**

Setter方法的名字以set开头，有一个方法参数。

setters方法有可能会有返回值也有可能没有，一些Setter方法返回void，一些用来设置值，有一些对象的setter方法在方法链中被调用（译者注：这类的setter方法必须要有返回值），因此你不应该妄自假设setter方法的返回值，一切应该视情况而定。

下面是一个获取getter方法和setter方法的例子：

````java
public static void printGettersSetters(Class aClass){
  Method[] methods = aClass.getMethods();
  for(Method method : methods){
    if(isGetter(method)) System.out.println("getter: " + method);
    if(isSetter(method)) System.out.println("setter: " + method);
  }
}
public static boolean isGetter(Method method){
  if(!method.getName().startsWith("get"))      return false;
  if(method.getParameterTypes().length != 0)   return false;
  if(void.class.equals(method.getReturnType()) return false;
  return true;
}
public static boolean isSetter(Method method){
  if(!method.getName().startsWith("set")) return false;
  if(method.getParameterTypes().length != 1) return false;
  return true;
}
````

### 私有变量和私有方法  

​	在通常的观点中从对象的外部访问私有变量以及方法是不允许的，但是Java反射机制可以做到这一点。使用这个功能并不困难，在进行单元测试时这个功能非常有效。本节会向你展示如何使用这个功能。

注意：这个功能只有在代码运行在单机Java应用`(standalone Java application)`中才会有效,就像你做单元测试或者一些常规的应用程序一样。如果你在Java Applet中使用这个功能，那么你就要想办法去应付`SecurityManager`对你限制了。但是一般情况下我们是不会这么做的，所以在本节里面我们不会探讨这个问题。

- 访问私有变量

  要想获取私有变量你可以调用`Class.getDeclaredField(String name)`方法或者`Class.getDeclaredFields()`方法。`Class.getField(String name)`和`Class.getFields()`只会返回公有的变量，无法获取私有变量。下面例子定义了一个包含私有变量的类，在它下面是如何通过反射获取私有变量的例子：

  ````java]
  public class PrivateObject {
    private String privateString = null;
    public PrivateObject(String privateString) {
      this.privateString = privateString;
    }
  }
  ````

  ````java
  PrivateObject privateObject = new PrivateObject("The Private Value");
  Field privateStringField = PrivateObject.class.getDeclaredField("privateString");
  privateStringField.setAccessible(true);
  String fieldValue = (String) privateStringField.get(privateObject);
  System.out.println("fieldValue = " + fieldValue);

  ````

  这个例子会输出`fieldValue = The Private Value`，`The Private Value`是`PrivateObject`实例的`privateString`私有变量的值，注意调用`PrivateObject.class.getDeclaredField(“privateString”)`方法会返回一个私有变量，这个方法返回的变量是定义在`PrivateObject`类中的而不是在它的父类中定义的变量。
  注意`privateStringField.setAccessible(true)`这行代码，通过调用`setAccessible()`方法会关闭指定类Field实例的反射访问检查，这行代码执行之后不论是私有的、受保护的以及包访问的作用域，你都可以在任何地方访问，即使你不在他的访问权限作用域之内。但是你如果你用一般代码来访问这些不在你权限作用域之内的代码依然是不可以的，在编译的时候就会报错。

- 访问私有方法

  访问一个私有方法你需要调用` Class.getDeclaredMethod(String name, Class[] parameterTypes)`或者`Class.getDeclaredMethods() `方法。 `Class.getMethod(String name, Class[] parameterTypes)`和`Class.getMethods()`方法，只会返回公有的方法，无法获取私有方法。下面例子定义了一个包含私有方法的类，在它下面是如何通过反射获取私有方法的例子：

  ````java
  public class PrivateObject {
    private String privateString = null;
    public PrivateObject(String privateString) {
      this.privateString = privateString;
    }
    private String getPrivateString(){
      return this.privateString;
    }
  }
  ````

  ````java
  PrivateObject privateObject = new PrivateObject("The Private Value");
  Method privateStringMethod = PrivateObject.class.
          getDeclaredMethod("getPrivateString", null);
  privateStringMethod.setAccessible(true);
  String returnValue = (String)
          privateStringMethod.invoke(privateObject, null);
  System.out.println("returnValue = " + returnValue);

  ````

  这个例子会输出`returnValue = The Private Value`，`The Private Value`是`PrivateObject`实例的`getPrivateString()`方法的返回值。
  `PrivateObject.class.getDeclaredMethod(“privateString”)`方法会返回一个私有方法，这个方法是定义在`PrivateObject`类中的而不是在它的父类中定义的。
  同样的，注意`Method.setAcessible(true)`这行代码，通过调用`setAccessible()`方法会关闭指定类的Method实例的反射访问检查，这行代码执行之后不论是私有的、受保护的以及包访问的作用域，你都可以在任何地方访问，即使你不在他的访问权限作用域之内。但是你如果你用一般代码来访问这些不在你权限作用域之内的代码依然是不可以的，在编译的时候就会报错。

  ​

### 注解  

利用Java反射机制可以在运行期获取Java类的注解信息。

- 什么是注解

  注解是Java 5的一个新特性。注解是插入你代码中的一种注释或者说是一种元数据（meta data）。这些注解信息可以在编译期使用预编译工具进行处理（pre-compiler tools），也可以在运行期使用Java反射机制进行处理。下面是一个类注解的例子：

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

### 泛型  

- 运用泛型反射的经验法则

  下面是两个典型的使用泛型的场景：
  1、声明一个需要被参数化（parameterizable）的类/接口。
  2、使用一个参数化类。

  当你声明一个类或者接口的时候你可以指明这个类或接口可以被参数化，`java.util.List`接口就是典型的例子。你可以运用泛型机制创建一个标明存储的是String类型list，这样比你创建一个Object的list要更好。

  当你想在运行期参数化类型本身，比如你想检查java.util.List类的参数化类型，你是没有办法能知道他具体的参数化类型是什么。这样一来这个类型就可以是一个应用中所有的类型。但是，当你检查一个使用了被参数化的类型的变量或者方法，你可以获得这个被参数化类型的具体参数。总之：

  你不能在运行期获知一个被参数化的类型的具体参数类型是什么，但是你可以在用到这个被参数化类型的方法以及变量中找到他们，换句话说就是获知他们具体的参数化类型。
  在下面的段落中会向你演示这类情况。

- 泛型方法返回类型

  如果你获得了`java.lang.reflect.Method`对象，那么你就可以获取到这个方法的泛型返回类型信息。如果方法是在一个被参数化类型之中那么你无法获取他的具体类型，但是如果方法返回一个泛型类那么你就可以获得这个泛型类的具体参数化类型。

  下面这个例子定义了一个类这个类中的方法返回类型是一个泛型类型：

  ````java
  public class MyClass {
    protected List<String> stringList = ...;
    public List<String> getStringList(){
      return this.stringList;
    }
  }
  ````

  我们可以获取`getStringList()`方法的泛型返回类型，换句话说，我们可以检测到`getStringList()`方法返回的是List而不仅仅只是一个List。如下例：

  ````java
  Method method = MyClass.class.getMethod("getStringList", null);
  Type returnType = method.getGenericReturnType();
  if(returnType instanceof ParameterizedType){
      ParameterizedType type = (ParameterizedType) returnType;
      Type[] typeArguments = type.getActualTypeArguments();
      for(Type typeArgument : typeArguments){
          Class typeArgClass = (Class) typeArgument;
          System.out.println("typeArgClass = " + typeArgClass);
      }
  }
  ````

  这段代码会打印出 `typeArgClass = java.lang.String`，`Type[]`数组`ypeArguments`只有一个结果 – 一个代表`java.lang.String`的`Class`类的实例。Class类实现了Type接口。

  ​

- 泛型方法参数类型

  你同样可以通过反射来获取方法参数的泛型类型，下面这个例子定义了一个类，这个类中的方法的参数是一个被参数化的List：

  ````java
  public class MyClass {
    protected List<String> stringList = ...;
    public void setStringList(List<String> list){
      this.stringList = list;
    }
  }
  ````

  你可以像这样来获取方法的泛型参数：

  ````java
  method = Myclass.class.getMethod("setStringList", List.class);
  Type[] genericParameterTypes = method.getGenericParameterTypes();
  for(Type genericParameterType : genericParameterTypes){
      if(genericParameterType instanceof ParameterizedType){
          ParameterizedType aType = (ParameterizedType) genericParameterType;
          Type[] parameterArgTypes = aType.getActualTypeArguments();
          for(Type parameterArgType : parameterArgTypes){
              Class parameterArgClass = (Class) parameterArgType;
              System.out.println("parameterArgClass = " + parameterArgClass);
          }
      }
  }
  ````

  这段代码会打印出`parameterArgType = java.lang.String`。`Type[]`数组`parameterArgTypes`只有一个结果 – 一个代表`java.lang.String`的Class类的实例。Class类实现了Type接口。

- 泛型变量类型

  同样可以通过反射来访问公有（Public）变量的泛型类型，无论这个变量是一个类的静态成员变量或是实例成员变量,例子，一个定义了一个名为stringList的成员变量的类。

  ````java
  public class MyClass {
    public List<String> stringList = ...;
  }
   
  Field field = MyClass.class.getField("stringList");
  Type genericFieldType = field.getGenericType();
  if(genericFieldType instanceof ParameterizedType){
      ParameterizedType aType = (ParameterizedType) genericFieldType;
      Type[] fieldArgTypes = aType.getActualTypeArguments();
      for(Type fieldArgType : fieldArgTypes){
          Class fieldArgClass = (Class) fieldArgType;
          System.out.println("fieldArgClass = " + fieldArgClass);
      }
  }
  ````

  这段代码会打印出`fieldArgClass = java.lang.String`。Type[]数组`eldArgClass`只有一个结果 – 一个代表`java.lang.String`的Class类的实例。Class类实现了Type接口。

### 数组  

​	利用反射机制来处理数组会有点棘手。尤其是当你想要获得一个数组的Class对象，比如int[]等等。本节会讨论通过反射机制创建数组和如何获取数组的Class对象。

- **java.lang.reflect.Array**

  Java反射机制通过`java.lang.reflect.Array`这个类来处理数组。不要把这个类与Java集合套件（`Collections suite`）中的`java.util.Arrays`混淆，`java.util.Arrays`是一个提供了遍历数组，将数组转化为集合等工具方法的类。

- 创建一个数组

  Java反射机制通过`java.lang.reflect.Array`类来创建数组。下面是一个如何创建数组的例子：

  ````java'
  int[] intArray = (int[]) Array.newInstance(int.class, 3);
  ````

  这个例子创建一个int类型的数组。Array.newInstance()方法的第一个参数表示了我们要创建一个什么类型的数组。第二个参数表示了这个数组的空间是多大。

- 访问一个数组

  通过Java反射机制同样可以访问数组中的元素。具体可以使用Array.get(…)和Array.set(…)方法来访问。下面是一个例子：

  ````java
  int[] intArray = (int[]) Array.newInstance(int.class, 3);
  Array.set(intArray, 0, 123);

  Array.set(intArray, 1, 456);

  Array.set(intArray, 2, 789);

  System.out.println("intArray[0] = " + Array.get(intArray, 0));

  System.out.println("intArray[1] = " + Array.get(intArray, 1));

  System.out.println("intArray[2] = " + Array.get(intArray, 2));


  这个例子会输出：
  intArray[0] = 123
  intArray[1] = 456
  intArray[2] = 789
  ````

- 获取数组的Class对象

  ````java
  Class stringArrayClass = String[].class;
  ````

  如果使用Class.forName()方法来获取Class对象则不是那么简单。比如你可以像这样来获得一个原生数据类型（primitive）int数组的Class对象：

  ````java
  Class intArray = Class.forName("[I");
  ````

  在JVM中字母I代表int类型，左边的‘[’代表我想要的是一个int类型的数组，这个规则同样适用于其他的原生数据类型。
  对于普通对象类型的数组有一点细微的不同：

  ````java
  Class stringArrayClass = Class.forName("[Ljava.lang.String;");
  ````

  注意‘[L’的右边是类名，类名的右边是一个‘;’符号。这个的含义是一个指定类型的数组。
  需要注意的是，你不能通过Class.forName()方法获取一个原生数据类型的Class对象。下面这两个例子都会报ClassNotFoundException：

  ````java
  Class intClass1 = Class.forName("I");
  Class intClass2 = Class.forName("int");
  ````

  我通常会用下面这个方法来获取普通对象以及原生对象的Class对象：

  ````java
  public Class getClass(String className){
    if("int" .equals(className)) return int .class;
    if("long".equals(className)) return long.class;
    return Class.forName(className);
  }

  ````

  ​