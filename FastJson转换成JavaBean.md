# FastJson对于JSON格式字符串、JSON对象及JavaBean之间的相互转换  

### 前言  

​	`fastJson`对于`json`格式字符串的解析主要用到了一下三个类：

​	`JSON：fastJson`的解析器，用于`JSON`格式字符串与`JSON`对象及`javaBean`之间的转换。

​	`JSONObject：fastJson`提供的`json`对象。

​	`JSONArray：fastJson`提供`json`数组对象。

- 我们可以把`JSONObject`当成一个`Map<String,Object>`来看，只是`JSONObject`提供了更为丰富便捷的方法，方便我们对于对象属性的操作。我们看一下源码。

  ```java
  public class JSONObject extends JSON implements Map<String, Object>, Cloneable, Serializable, InvocationHandler {

      private static final long         serialVersionUID         = 1L;
      private static final int          DEFAULT_INITIAL_CAPACITY = 16;

      private final Map<String, Object> map;

      public JSONObject(){
          this(DEFAULT_INITIAL_CAPACITY, false);
      }

      public JSONObject(Map<String, Object> map){
          this.map = map;
      }

      public JSONObject(boolean ordered){
          this(DEFAULT_INITIAL_CAPACITY, ordered);
      }

      public JSONObject(int initialCapacity){
          this(initialCapacity, false);
      }

      public JSONObject(int initialCapacity, boolean ordered){
          if (ordered) {
              map = new LinkedHashMap<String, Object>(initialCapacity);
          } else {
              map = new HashMap<String, Object>(initialCapacity);
          }
      }
    ...
  ```

- 同样我们可以把JSONArray当做一个List<Object>，可以把JSONArray看成JSONObject对象的一个集合。


