# Java5 新特性

#### 特性概览

- 泛型
- 枚举
- 装箱拆箱
- 变长参数
- 注解
- `foreach`循环
- 静态导入
- 格式化
- 线程框架/数据结构
- `Arrays`工具类/`StringBuilder`/`instrument`

#### 具体内容

- 泛型

  所谓类型擦除指的就是Java源码中的范型信息只允许停留在编译前期，而编译后的字节码文件中将不再保留任何的范型信息。也就是说，范型信息在编译时将会被全部删除，其中范型类型的类型参数则会被替换为`Object`类型，并在实际使用时强制转换为指定的目标数据类型。而C++中的模板则会在编译时将模板类型中的类型参数根据所传递的指定数据类型生成相对应的目标代码。

  引用泛型之后，允许指定集合里元素的类型，免去了强制类型转换，并且能在编译时刻进行类型检查的好处。

  ``Parameterized Type``作为参数和返回值，``Generic``是``vararg``、``annotation``、``enumeration``、``collection``的基石。

  - 类型安全

   抛弃`List、Map`，使用`List<T>、Map<K,V>`给它们添加元素或者使用`Iterator<T>`遍历时，编译期就可以给你检查出类型错误

  - 方法参数和返回值加上了Type
    抛弃`List、Map`，使用`List<T>、Map<K,V>`

  - 不需要类型转换

      `List<String> list=new ArrayList<String>();`

  ​         `String str=list.get(i);`
  - 类型通配符“?”

     假设一个打印`List<T>`中元素的方法`printList`,我们希望任何类型`T`的`List<T>`都可以被打印：

  ```java
  public void printList(List<?> list,PrintStream out)throws IOException{  
    for(Iterator<?> i=list.iterator();i.hasNext();){  
      System.out.println(i.next.toString());  
    }  
  ```

  ​      如果通配符?让我们的参数类型过于广泛，我们可以把`List<?>`、`Iterator<?>` 修改为

  ​      `List<? Extends Number>、Iterator<? Extends Number>`限制一下它。

- 枚举

  - EnumMap

    ```java
    public void testEnumMap(PrintStream out) throws IOException {
        // Create a map with the key and a String message
        EnumMap<AntStatus, String> antMessages =
          new EnumMap<AntStatus, String>(AntStatus.class);
        // Initialize the map
        antMessages.put(AntStatus.INITIALIZING, "Initializing Ant...");
        antMessages.put(AntStatus.COMPILING,    "Compiling Java classes...");
        antMessages.put(AntStatus.COPYING,      "Copying files...");
        antMessages.put(AntStatus.JARRING,      "JARring up files...");
        antMessages.put(AntStatus.ZIPPING,      "ZIPping up files...");
        antMessages.put(AntStatus.DONE,         "Build complete.");
        antMessages.put(AntStatus.ERROR,        "Error occurred.");
        // Iterate and print messages
        for (AntStatus status : AntStatus.values() ) {
          out.println("For status " + status + ", message is: " +
                      antMessages.get(status));
        }
      }
    ```

  - switch枚举

    ```java
    public String getDescription() {
        switch(this) {
          case ROSEWOOD:      return "Rosewood back and sides";
          case MAHOGANY:      return "Mahogany back and sides";
          case ZIRICOTE:      return "Ziricote back and sides";
          case SPRUCE:        return "Sitka Spruce top";
          case CEDAR:         return "Wester Red Cedar top";
          case AB_ROSETTE:    return "Abalone rosette";
          case AB_TOP_BORDER: return "Abalone top border";
          case IL_DIAMONDS:   
            return "Diamonds and squares fretboard inlay";
          case IL_DOTS:
            return "Small dots fretboard inlay";
          default: return "Unknown feature";
        }
      }
    ```

- 自动拆装箱（`Autoboxing`与`Unboxing`）

  ​	将`primitive`类型转换成对应的`wrapper`类型：`Boolean、Byte、Short、Character、Integer、Long、Float、Double`。

  ```java
  public static void m1(Integer i){
          System.out.println("this is integer");
      }
      public static void m1(double d){
          System.out.println("this is double");
      }
  ```

- 变长参数

  ```java
  private String print(Object... values) {
      StringBuilder sb = new StringBuilder();
      for (Object o : values) {
        sb.append(o.toString())
          .append(" ");
      }
      return sb.toString();
    }
  ```

- 注解

- `foreach`循环

  `for/in`循环办不到的事情：

  - 遍历同时获取index
  - 集合逗号拼接时去掉最后一个
  - 遍历的同时删除元素	

- 静态导入

  ```java'
  import static java.lang.System.err;
  import static java.lang.System.out;
  import java.io.IOException;
  import java.io.PrintStream;
  public class StaticImporter {
    public static void writeError(PrintStream err, String msg) 
      throws IOException {
     
      // Note that err in the parameter list overshadows the imported err
      err.println(msg); 
    }
    public static void main(String[] args) {
      if (args.length < 2) {
        err.println(
          "Incorrect usage: java com.oreilly.tiger.ch08 [arg1] [arg2]");
        return;
      }
      out.println("Good morning, " + args[0]);
      out.println("Have a " + args[1] + " day!");
      try {
        writeError(System.out, "Error occurred.");
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
  }

  ```

- 格式化

  ```java
  /**
   * java.text.DateFormat
   * java.text.SimpleDateFormat
   * java.text.MessageFormat
   * java.text.NumberFormat
   * java.text.ChoiceFormat
   * java.text.DecimalFormat
   */
  public class FormatTester {
      public static void printf() {
          //printf
          String filename = "this is a file";
          try {
              File file = new File(filename);
              FileReader fileReader = new FileReader(file);
              BufferedReader reader = new BufferedReader(fileReader);
              String line;
              int i = 1;
              while ((line = reader.readLine()) != null) {
                  System.out.printf("Line %d: %s%n", i++, line);
              }
          } catch (Exception e) {
              System.err.printf("Unable to open file named '%s': %s",
                      filename, e.getMessage());
          }
      }
      public static void stringFormat() {
          // Format a string containing a date.
          Calendar c = new GregorianCalendar(1995, MAY, 23);
          String s = String.format("Duke's Birthday: %1$tm %1$te,%1$tY", c);
          // -> s == "Duke's Birthday: May 23, 1995"
          System.out.println(s);
      }
      public static void formatter() {
          StringBuilder sb = new StringBuilder();
          // Send all output to the Appendable object sb
          Formatter formatter = new Formatter(sb, Locale.US);
          // Explicit argument indices may be used to re-order output.
          formatter.format("%4$2s %3$2s %2$2s %1$2s", "a", "b", "c", "d");
          // -> " d  c  b  a"
          // Optional locale as the first argument can be used to get
          // locale-specific formatting of numbers.  The precision and width can be
          // given to round and align the value.
          formatter.format(Locale.FRANCE, "e = %+10.4f", Math.E);
          // -> "e =    +2,7183"
          // The '(' numeric flag may be used to format negative numbers with
          // parentheses rather than a minus sign.  Group separators are
          // automatically inserted.
          formatter.format("Amount gained or lost since last statement: $ %(,.2f",
                  6217.58);
          // -> "Amount gained or lost since last statement: $ (6,217.58)"
      }
      public static void messageFormat() {
          String msg = "欢迎光临，当前（{0}）等待的业务受理的顾客有{1}位，请排号办理业务！";
          MessageFormat mf = new MessageFormat(msg);
          String fmsg = mf.format(new Object[]{new Date(), 35});
          System.out.println(fmsg);
      }
      public static void dateFormat(){
          String str = "2010-1-10 17:39:21";
          SimpleDateFormat format = new SimpleDateFormat("yyyyMMddHHmmss");
          try {
              System.out.println(format.format(format.parse(str)));
          } catch (ParseException e) {
              // TODO Auto-generated catch block
              e.printStackTrace();
          }
      }
      public static void main(String[] args) {
          formatter();
          stringFormat();
          messageFormat();
          dateFormat();
          printf();
      }
  }
  ```

- 线程框架/数据结构

  - `uncaught exception` 未捕获的异常

    ```java
    public class BubbleSortThread extends Thread {
      private int[] numbers;
      public BubbleSortThread(int[] numbers) {
        setName("Simple Thread");
        setUncaughtExceptionHandler(
          new SimpleThreadExceptionHandler());
        this.numbers = numbers;
      }
      public void run() {
        int index = numbers.length;
        boolean finished = false;
        while (!finished) {
          index--;
          finished = true;
          for (int i=0; i<index; i++) {
            // Create error condition
            if (numbers[i+1] < 0) {
              throw new IllegalArgumentException(
                "Cannot pass negative numbers into this thread!");
            }
            if (numbers[i] > numbers[i+1]) {
              // swap
              int temp = numbers[i];
              numbers[i] = numbers[i+1];
              numbers[i+1] = temp;
              finished = false;
            }
          }
        }    
      }
    }
    class SimpleThreadExceptionHandler implements
        Thread.UncaughtExceptionHandler {
      public void uncaughtException(Thread t, Throwable e) {
        System.err.printf("%s: %s at line %d of %s%n",
            t.getName(), 
            e.toString(),
            e.getStackTrace()[0].getLineNumber(),
            e.getStackTrace()[0].getFileName());
      }
    }
    ```

  - `blocking queue` 阻塞队列

    ```java
    public class Producer extends Thread {
      private BlockingQueue q;
      private PrintStream out;
      public Producer(BlockingQueue q, PrintStream out) {
        setName("Producer");
        this.q = q;
        this.out = out;
      }
      public void run() {
        try {
          while (true) {
            q.put(produce());
          }
        } catch (InterruptedException e) {
          out.printf("%s interrupted: %s", getName(), e.getMessage());
        }
      }
      private String produce() {
        while (true) {
          double r = Math.random();
          // Only goes forward 1/10 of the time
          if ((r*100) < 10) {
            String s = String.format("Inserted at %tc", new Date());
            return s;
          }
        }
      }
    }
    ```

  - 线程池

    著名的JUC类库。

    - 每次提交任务时，如果线程数还没达到`coreSize`就创建新线程并绑定该任务。 所以第`coreSize`次提交任务后线程总数必达到`coreSize`，不会重用之前的空闲线程。
    - 线程数达到`coreSize`后，新增的任务就放到工作队列里，而线程池里的线程则努力的使用`take()`从工作队列里拉活来干。
    - 如果队列是个有界队列，又如果线程池里的线程不能及时将任务取走，工作队列可能会满掉，插入任务就会失败，此时线程池就会紧急的再创建新的临时线程来补救。
    - 临时线程使用`poll(keepAliveTime，timeUnit)`来从工作队列拉活，如果时候到了仍然两手空空没拉到活，表明它太闲了，就会被解雇掉。
    - 如果`core`线程数＋临时线程数 >`maxSize`，则不能再创建新的临时线程了，转头执行`RejectExecutionHanlder`。默认的`AbortPolicy`抛`RejectedExecutionException`异常，其他选择包括静默放弃当前任务`(Discard)`，放弃工作队列里最老的任务`(DisacardOldest)`，或由主线程来直接执行`(CallerRuns)` ，或你自己发挥想象力写的一个。

- 其他

  - Arrays

    ```java
    Arrays.sort(myArray);
    Arrays.toString(myArray)
    Arrays.binarySearch(myArray, 98)
    Arrays.deepToString(ticTacToe)
    Arrays.deepEquals(ticTacToe, ticTacToe3)
    ```

  - Queue

    避开集合的`add/remove`操作，使用`offer、poll`操作（不抛异常）

    ```java
    Queue q = new LinkedList(); //采用它来实现queue
    ```

  - `Override`返回类型
    支持协变返回

  - 单线程`StringBuilder`
    线程不安全，在单线程下替换string buffer提高性能

  - `java.lang.instrument`