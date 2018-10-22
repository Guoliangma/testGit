# Java6 新特性

#### 特性概览

- JSR223脚本引擎
- JSR199--Java Compiler API
- JSR269--Pluggable Annotation Processing API
- 支持JDBC4.0规范
- JAX-WS 2.0规范

#### 具体内容

- ### JSR223脚本引擎

  Scripting for the Java Platform。

  - 基本使用

   ```java
  public class BasicScripting {
      public void greet() throws ScriptException {
          ScriptEngineManager manager = new ScriptEngineManager();
          //支持通过名称、文件扩展名、MIMEtype查找
          ScriptEngine engine = manager.getEngineByName("JavaScript");
  //        engine = manager.getEngineByExtension("js");
  //        engine = manager.getEngineByMimeType("text/javascript");
          if (engine == null) {
              throw new RuntimeException("找不到JavaScript语言执行引擎。");
          }
          engine.eval("println('Hello!');");
      }
      public static void main(String[] args) {
          try {
              new BasicScripting().greet();
          } catch (ScriptException ex) {
              Logger.getLogger(BasicScripting.class.getName()).log(Level.SEVERE, null, ex);
          }
      }
  }
   ```

  - 绑定上下文

    ```java
    public class ScriptContextBindings extends JsScriptRunner {
        public void scriptContextBindings() throws ScriptException {
            ScriptEngine engine = getJavaScriptEngine();
            ScriptContext context = engine.getContext();
            Bindings bindings1 = engine.createBindings();
            bindings1.put("name", "Alex");
            context.setBindings(bindings1, ScriptContext.GLOBAL_SCOPE);
            Bindings bindings2 = engine.createBindings();
            bindings2.put("name", "Bob");
            context.setBindings(bindings2, ScriptContext.ENGINE_SCOPE);
            engine.eval("println(name);");
        }
        public void useScriptContextValues() throws ScriptException {
            ScriptEngine engine = getJavaScriptEngine();
            ScriptContext context = engine.getContext();
            Bindings bindings = context.getBindings(ScriptContext.ENGINE_SCOPE);
            bindings.put("name", "Alex");
            engine.eval("println(name);");
        }
        public void attributeInBindings() throws ScriptException {
            ScriptEngine engine = getJavaScriptEngine();
            ScriptContext context = engine.getContext();
            context.setAttribute("name", "Alex", ScriptContext.GLOBAL_SCOPE);
            engine.eval("println(name);");
        }
        /**
         * @param args the command line arguments
         */
        public static void main(String[] args) throws ScriptException {
            ScriptContextBindings scb = new ScriptContextBindings();
            scb.scriptContextBindings();
            scb.useScriptContextValues();
            scb.attributeInBindings();
        }
    }
    ```

- ### JSR199--Java Compiler API

  ```java
  public class JavaCompilerAPICompiler {
      public void compile(Path src, Path output) throws IOException {
          JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
          try (StandardJavaFileManager fileManager = compiler.getStandardFileManager(null, null, null)) {
              Iterable<? extends JavaFileObject> compilationUnits = fileManager.getJavaFileObjects(src.toFile());
              Iterable<String> options = Arrays.asList("-d", output.toString());
              JavaCompiler.CompilationTask task = compiler.getTask(null, fileManager, null, options, null, compilationUnits);
              boolean result = task.call();
          }
      }
  }
  ```

- ### JSR269--Pluggable Annotation Processing API

  一部分是进行注解处理的`javax.annotation.processing`，另一部分是对程序的静态结构进行建模的`javax.lang.model`

- 支持JDBC4.0规范

- JAX-WS 2.0规范(`包括JAXB 2.0`)

- 轻量级HttpServer

  ​

# Java7 新特性

#### 特性概览

- switch中支持字符串
- 泛型实例化类型自动推断
- 语法上支持集合，而不一定是数组
- 新增一些取环境信息的工具方法
- Boolean类型反转，空指针安全,参与位运算
- 两个char间的equals 
- 安全的加减乘除 
- map集合支持并发请求

#### 具体内容

- switch中支持字符串

  ```java'
  String s = "test"; 
  switch (s) { 
  	case "test" : 
  		System.out.println("test"); 
  	case "test1" : 
  		System.out.println("test1"); 
  	break ; 
  	default : 
  		System.out.println("break"); 
  	break ; 
  }
  ```

- 泛型实例化类型自动推断

  ```java
  运用List tempList = new ArrayList<>(); 
  ```

- 语法上支持集合，而不一定是数组

  ```java
  final List piDigits = [ 1,2,3,4,5,8 ];
  ```

- 新增一些取环境信息的工具方法

  - File

    ```java
    File System.getJavaIoTempDir() // IO临时文件夹
    File System.getJavaHomeDir() // JRE的安装目录
    File System.getUserHomeDir() // 当前用户目录
    File System.getUserDir() // 启动java进程时所在的目录5
    ```

- Boolean类型反转，空指针安全,参与位运算

  ```java
  Boolean Booleans.negate(Boolean booleanObj)
  True => False , False => True, Null => Null
  boolean Booleans.and(boolean[] array)
  boolean Booleans.or(boolean[] array)
  boolean Booleans.xor(boolean[] array)
  boolean Booleans.and(Boolean[] array)
  boolean Booleans.or(Boolean[] array)
  boolean Booleans.xor(Boolean[] array)
  ```

- 两个char间的equals 

  ```java
  boolean Character.equalsIgnoreCase(char ch1, char ch2)
  ```

- 安全的加减乘除 

  ```java
  int Math.safeToInt(long value)
  int Math.safeNegate(int value)
  long Math.safeSubtract(long value1, int value2)
  long Math.safeSubtract(long value1, long value2)
  int Math.safeMultiply(int value1, int value2)
  long Math.safeMultiply(long value1, int value2)
  long Math.safeMultiply(long value1, long value2)
  long Math.safeNegate(long value)
  int Math.safeAdd(int value1, int value2)
  long Math.safeAdd(long value1, int value2)
  long Math.safeAdd(long value1, long value2)
  int Math.safeSubtract(int value1, int value2)
  ```

- map集合支持并发请求，且可以写成 Map map = {name:"xxx",age:18};