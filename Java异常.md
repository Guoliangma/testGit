# Java 异常

- 什么是Java异常

  异常类分两大类型：Error类代表了编译和系统的错误，不允许捕获；Exception类代表了标准Java库方法所激发的异常。Exception类还包含运行异常类``Runtime_Exception``和非运行异常类``Non_RuntimeException``这两个直接的子类。

  运行异常类对应于编译错误，它是指Java程序在运行时产生的由解释器引发的各种异常。运行异常可能出现在任何地方，且出现频率很高，因此为了避免巨大的系统资源开销，编译器不对异常进行检查。所以Java语言中的运行异常不一定被捕获。出现运行错误往往表示代码有错误，如：算数异常（如被0除）、下标异常（如数组越界）等。

  非运行异常时``Non_RuntimeException``类及其子类的实例，又称为可检测异常。Java编译器利用分析方法或构造方法中可能产生的结果来检测Java程序中是否含有检测异常的处理程序，对于每个可能的可检测异常，方法或构造方法的throws子句必须列出该异常对应的类。在Java的标准包``java.lang java.util`` 和 ``java.net ``中定义的异常都是非运行异常。  

- 常见异常

  算术异常类：``ArithmeticExecption``

  空指针异常类：``NullPointerException``

  类型强制转换异常：``ClassCastException``

  数组负下标异常：``NegativeArrayException``

  数组下标越界异常：``ArrayIndexOutOfBoundsException``

  违背安全原则异常：``SecturityException``

  文件已结束异常：``EOFException``

  文件未找到异常：``FileNotFoundException``

  字符串转换为数字异常：``NumberFormatException``

  操作数据库异常：``SQLException``

  输入输出异常：``IOException``

  方法未找到异常：``NoSuchMethodException``

- Java异常类层次图：

  ![HashMap数据结构](http://img.my.csdn.net/uploads/201211/27/1354020417_5176.jpg)

  在 Java 中，所有的异常都有一个共同的祖先 `Throwable`（可抛出）。`Throwable` 指定代码中可用异常传播机制通过 Java 应用程序传输的任何问题的共性。
  ​       **Throwable：** 有两个重要的子类：``Exception``（异常）和 ``Error``（错误），二者都是 Java 异常处理的重要子类，各自都包含大量子类。

  ​       **Error（错误）:**是程序无法处理的错误，表示运行应用程序中较严重问题。大多数错误与代码编写者执行的操作无关，而表示代码运行时 JVM（Java 虚拟机）出现的问题。例如，Java虚拟机运行错误（``Virtual MachineError``），当 JVM 不再有继续执行操作所需的内存资源时，将出现 ``OutOfMemoryError``。这些异常发生时，Java虚拟机（JVM）一般会选择线程终止。这些错误表示故障发生于虚拟机自身、或者发生在虚拟机试图执行应用时，如Java虚拟机运行错误（``Virtual MachineError``）、类定义错误（``NoClassDefFoundError``）等。这些错误是不可查的，因为它们在应用程序的控制和处理能力之 外，而且绝大多数是程序运行时不允许出现的状况。对于设计合理的应用程序来说，即使确实发生了错误，本质上也不应该试图去处理它所引起的异常状况。在 Java中，错误通过Error的子类描述。

  ​       **Exception（异常）:**是程序本身可以处理的异常。

  ​       Exception 类有一个重要的子类 ``RuntimeException``。``RuntimeException ``类及其子类表示“JVM 常用操作”引发的错误。例如，若试图使用空值对象引用、除数为零或数组越界，则分别引发运行时异常（``NullPointerException``、``ArithmeticException``）和 ``ArrayIndexOutOfBoundException``。

     注意：异常和错误的区别：异常能被程序本身可以处理，错误是无法处理。

     通常，Java的异常(包括``Exception``和``Error``)分为**可查的异常（checked exceptions）和不可查的异常（unchecked exceptions）**。
  ​      可查异常（编译器要求必须处置的异常）：正确的程序在运行中，很容易出现的、情理可容的异常状况。可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常状况，就必须采取某种方式进行处理。

  ​      除了``RuntimeException``及其子类以外，其他的``Exception``类及其子类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。

  ​     不可查异常(编译器不要求强制处置的异常):包括运行时异常（``RuntimeException``与其子类）和错误（Error）。

  ​     Exception 这种异常分两大类运行时异常和非运行时异常(编译异常)。程序中应当尽可能去处理这些异常。

  ​       **运行时异常：**都是``RuntimeException``类及其子类异常，如``NullPointerException``(空指针异常)、``IndexOutOfBoundsException``(下标越界异常)等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

  ​      运行时异常的特点是Java编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用try-catch语句捕获它，也没有用throws子句声明抛出它，也会编译通过。
  ​       **非运行时异常 （编译异常）：**是``RuntimeException``以外的异常，类型上都属于``Exception``类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如``IOException``、``SQLException``等以及用户自定义的``Exception``异常，一般情况下不自定义检查异常。

-  处理异常机制

  ​	在 Java 应用程序中，异常处理机制为：抛出异常，捕捉异常。

  ​	**抛出异常 **：当一个方法出现错误引发异常时，方法创建异常对象并交付运行时系统，异常对象中包含了异常类型和异常出现时的程序状态等异常信息。运行时系统负责寻找处置异常的代码并执行。

  ​       **捕获异常**：在方法抛出异常之后，运行时系统将转为寻找合适的异常处理器（``exception handler``）。潜在的异常处理器是异常发生时依次存留在调用栈中的方法的集合。当异常处理器所能处理的异常类型与方法抛出的异常类型相符时，即为合适 的异常处理器。运行时系统从发生异常的方法开始，依次回查调用栈中的方法，直至找到含有合适异常处理器的方法并执行。当运行时系统遍历调用栈而未找到合适 的异常处理器，则运行时系统终止。同时，意味着Java程序的终止。

  ​        对于运行时异常、错误或可查异常，Java技术所要求的异常处理方式有所不同。

  ​        由于运行时异常的不可查性，为了更合理、更容易地实现应用程序，Java规定，运行时异常将由Java运行时系统自动抛出，允许应用程序忽略运行时异常。

  ​       对于方法运行中可能出现的Error，当运行方法不欲捕捉时，Java允许该方法不做任何抛出声明。因为，大多数Error异常属于永远不能被允许发生的状况，也属于合理的应用程序不该捕捉的异常。

  ​       对于所有的可查异常，Java规定：一个方法必须捕捉，或者声明抛出方法之外。也就是说，当一个方法选择不捕捉可查异常时，它必须声明将抛出异常。

  ​        能够捕捉异常的方法，需要提供相符类型的异常处理器。所捕捉的异常，可能是由于自身语句所引发并抛出的异常，也可能是由某个调用的方法或者Java运行时 系统等抛出的异常。也就是说，一个方法所能捕捉的异常，一定是Java代码在某处所抛出的异常。简单地说，异常总是先被抛出，后被捕捉的。

  ​         任何Java代码都可以抛出异常，如：自己编写的代码、来自Java开发环境包中代码，或者Java运行时系统。无论是谁，都可以通过Java的throw语句抛出异常。

  ​        从方法中抛出的任何异常都必须使用throws子句。

  ​        捕捉异常通过try-catch语句或者try-catch-finally语句实现。

  ​         总体来说，Java规定：对于可查异常必须捕捉、或者声明抛出。允许忽略不可查的``RuntimeException``和``Error``。  

- try-catch语句

  ```java
  try {  
      // 可能会发生异常的程序代码  
  } catch (Type1 id1){  
      // 捕获并处置try抛出的异常类型Type1  
  }  
  catch (Type2 id2){  
       //捕获并处置try抛出的异常类型Type2  
  }  finally {  
      // 无论是否发生异常，都将执行的语句块  
  }  
  ```
  ​	 关键词try后的一对大括号将一块可能发生异常的代码包起来，称为监控区域。Java方法在运行过程中出现异常，则创建异常对象。将异常抛出监控区域之 外，由Java运行时系统试图寻找匹配的catch子句以捕获异常。若有匹配的catch子句，则运行其异常处理代码，try-catch语句结束。

  ​       匹配的原则是：如果抛出的异常对象属于catch子句的异常类，或者属于该异常类的子类，则认为生成的异常对象与catch块捕获的异常类型相匹配。

  ​	finally 块：无论是否捕获或处理异常，finally块里的语句都会被执行。当在try块或catch块中遇到return语句时，finally语句块将在方法返回之前被执行。在以下4种特殊情况下，finally块不会被执行：

  ​	1）在finally语句块中发生了异常。
  ​	2）在前面的代码中用了System.exit()退出程序。
  ​	3）程序所在的线程死亡。
  ​	4）关闭CPU。  

- #### try-catch-finally 规则

  1)  必须在 try 之后添加 catch 或 finally 块。try 块后可同时接 catch 和 finally 块，但至少有一个块。
  2) 必须遵循块顺序：若代码同时使用 catch 和 finally 块，则必须将 catch 块放在 try 块之后。
  3) catch 块与相应的异常类的类型相关。
  4) 一个 try 块可能有多个 catch 块。若如此，则执行第一个匹配块。即Java虚拟机会把实际抛出的异常对象依次和各个catch代码块声明的异常类型匹配，如果异常对象为某个异常类型或其子类的实例，就执行这个catch代码块，不会再执行其他的 catch代码块
  5) 可嵌套 try-catch-finally 结构。
  6) 在 try-catch-finally 结构中，可重新抛出异常。
  7) 除了下列情况，总将执行 finally 做为结束：JVM 过早终止（调用 System.exit(int)）；在 finally 块中抛出一个未处理的异常；计算机断电、失火、或遭遇病毒攻击。  

- #### **try、catch、finally语句块的执行顺序:**

  1)当try没有捕获到异常时：try语句块中的语句逐一被执行，程序将跳过catch语句块，执行finally语句块和其后的语句；

  2)当try捕获到异常，catch语句块里没有处理此异常的情况：当try语句块里的某条语句出现异常时，而没有处理此异常的catch语句块时，此异常将会抛给JVM处理，finally语句块里的语句还是会被执行，但finally语句块后的语句不会被执行；

  3)当try捕获到异常，catch语句块里有处理此异常的情况：在try语句块中是按照顺序来执行的，当执行到某一条语句出现异常时，程序将跳到catch语句块，并与catch语句块逐一匹配，找到与之对应的处理程序，其他的catch语句块将不会被执行，而try语句块中，出现异常之后的语句也不会被执行，catch语句块执行完后，执行finally语句块里的语句，最后执行finally语句块后的语句；  

- 图示try、catch、finally语句块的执行

![HashMap数据结构](http://img.my.csdn.net/uploads/201211/27/1354022670_6403.jpg)

- 抛出异常

  ​    任何Java代码都可以抛出异常，如：自己编写的代码、来自Java开发环境包中代码，或者Java运行时系统。无论是谁，都可以通过Java的throw语句抛出异常。从方法中抛出的任何异常都必须使用throws子句。

  - throws抛出异常

    如果一个方法可能会出现异常，但没有能力处理这种异常，可以在方法声明处用throws子句来声明抛出异常。例如汽车在运行时可能会出现故障，汽车本身没办法处理这个故障，那就让开车的人来处理。

    ​     throws语句用在方法定义时声明该方法要抛出的异常类型，如果抛出的是Exception异常类型，则该方法被声明为抛出所有的异常。多个异常可使用逗号分割。throws语句的语法格式为：

    ```java
    methodname throws Exception1,Exception2,..,ExceptionN  
    {  
    } 
    ```

    方法名后的throws Exception1,Exception2,...,ExceptionN 为声明要抛出的异常列表。当方法抛出异常列表的异常时，方法将不对这些类型及其子类类型的异常作处理，而抛向调用该方法的方法，由他去处理。例如：

    ```java
    public class TestException {  
        static void pop() throws NegativeArraySizeException {  
            // 定义方法并抛出NegativeArraySizeException异常  
            int[] arr = new int[-3]; // 创建数组  
        }  
      
        public static void main(String[] args) { // 主方法  
            try { // try语句处理异常信息  
                pop(); // 调用pop()方法  
            } catch (NegativeArraySizeException e) {  
                System.out.println("pop()方法抛出的异常");// 输出异常信息  
            }  
        }  
    ```

    ​	使用throws关键字将异常抛给调用者后，如果调用者不想处理该异常，可以继续向上抛出，但最终要有能够处理该异常的调用者。

    ​    pop方法没有处理异常NegativeArraySizeException，而是由main函数来处理。

- Throws抛出异常的规则

  1) 如果是不可查异常（``checked exception``即``Error``、``ntimeException``们的子类，那么可以不使用throws关键字来声明要抛出的异常，编译仍能顺利通过，但在运行时会被系统抛出。

  2）必须声明方法可抛出的任何可查异常（``checked exception``）。即如果一个方法可能出现受可查异常，要么用``try-catch``语句捕获，要么用``throws``子句声明将它抛出，否则会导致编译错误

  3)仅当抛出了异常，该方法的调用者才必须处理或者重新抛出该异常。当方法的调用者无力处理该异常的时候，应该继续抛出，而不是囫囵吞枣。

  4）调用方法必须遵循任何可查异常的处理和声明规则。若覆盖一个方法，则不能声明与覆盖方法不同的异常。声明的任何异常必须是被覆盖方法所声明异常的同类或子类。