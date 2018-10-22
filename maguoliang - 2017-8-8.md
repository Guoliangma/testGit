# 四川高院绩效考核系统代码复查  

## 具体问题

### 1、循环中查询SQL   

- 方法：`public Object jqButton85903_onClickServer(Item item)`

- 代码：

  ```java
  Iterator<JSONObject> ajiter = array.iterator();
  while (ajiter.hasNext()) {
    JSONObject temp = ajiter.next();
    TYjptSpyjbcAjpcAj pcaj = new TYjptSpyjbcAjpcAj();
    TYjptFj fj = new TYjptFj();
    Map<String, Object> aj = ajclAtyDao.getAjByAh("\'" + 	          	 String.valueOf(temp.get("ah") + "\'"));
    setAjpcajObject(pcaj, temp, aj, ajpc.getcBh(), fj);
    pcajList.add(pcaj);
    fjList.add(fj);
  }
  ```

- 问题：1、该方法查询案件时候采用的是案号直接查询虽然案号也是唯一的，但是在存在脏数据的表中会查询到多条案件信息，导致报错。2、数据量为2W的情况，使用案件编号查询比使用案号查询速度快70倍左右。3、循环中查询SQL。

- 改进：

  ```java
  //遍历案件并把案件编号储存
  Iterator<JSONObject> ajiterClah = array.iterator();
  List<Long> ajbhs = new ArrayList<>();
  while (ajiterClah.hasNext()) {
    JSONObject temp = ajiterClah.next();
    if (null != temp.get("ajbh")) {
      Long ajbh = Long.parseLong(String.valueOf(temp.get("ajbh")));
      ajbhs.add(ajbh);
    }
  }
  //根据案件编号List查询所有评查案件相关的案件信息
  List<TSpsjAjUnionall> ajList = ajUnionallMapper.getAjListByAjbhs(ajbhs);
  //遍历添加的案件，并封装添加案件评查的数据
  Iterator<JSONObject> ajiter = array.iterator();
  while (ajiter.hasNext()) {
    JSONObject temp = ajiter.next();
    TDataZlpc pcaj = new TDataZlpc();
    TSpsjAjUnionall aj = new TSpsjAjUnionall();
    for (TSpsjAjUnionall ajTemp : ajList) {
      if (ajTemp.getCAh().equals((temp).get("ah"))) {
        aj = ajTemp;
        break;
      }
    }
    setAjpcajObject(pcaj, temp, aj, zlpcbh);
    pcajList.add(pcaj);
  }                          
  ```

- 提出人：马国梁

- 提出时间：2018-8-8

### 2、Logic类职责不清

- 方法：`public Object jqButton85903_onClickServer(Item item)`

- 代码：

  ```java
  /**
       * 点击时脚本 保存
       * 
       * @param item
       *            控件对象
       * @throws ParseException
       */
      @SuppressWarnings("unchecked")
      public Object jqButton85903_onClickServer(Item item) throws ParseException {
          //案件评查编号
          String ajpcbh = ArteryParamUtil.getString("ajpcbh");
          //json封装的页面所有的案件信息和附件信息
          Object ajxinxi = ArteryParamUtil.getString("ajJson");
          JSONArray array = JSONArray.fromObject(ajxinxi);
          Map<String, Object> result = new HashMap<String, Object>();
          ArrayList<TYjptSpyjbcAjpcAj> pcajList = new ArrayList<TYjptSpyjbcAjpcAj>();
          ArrayList<TYjptFj> fjList = new ArrayList<TYjptFj>();

          TYjptSpyjbcAjpc ajpc = null;
          if (StringUtils.isEmpty(ajpcbh)) {
              ajpc = new TYjptSpyjbcAjpc();
              setAjpcObject(ajpc);
          } else {
              ajpc = ajpcService.getAjpcByAjpcbh(ajpcbh);
          }
          Iterator<JSONObject> ajiter = array.iterator();
          while (ajiter.hasNext()) {
              JSONObject temp = ajiter.next();
              TYjptSpyjbcAjpcAj pcaj = new TYjptSpyjbcAjpcAj();
              TYjptFj fj = new TYjptFj();
              Map<String, Object> aj = ajclAtyDao.getAjByAh("\'" + String.valueOf(temp.get("ah") + "\'"));
              setAjpcajObject(pcaj, temp, aj, ajpc.getcBh(), fj);
              pcajList.add(pcaj);
              fjList.add(fj);
          }

          TransactionStatus status = transaction.start();
          try {
              if (StringUtils.isEmpty(ajpcbh)) {
                  ajpcService.insertAjpc(ajpc);
                  ajpcService.insertAjpcajList(pcajList);
              } else {
                  ajpc.setDUpdate(DateUtil.getCurrDate());
                  ajpcService.updateAjpc(ajpc);
                  ajpcService.deleteAjpcajByAjpcbh(ajpcbh);
                  ajpcService.insertAjpcajList(pcajList);
              }
              for (TYjptFj fj : fjList) {
                  String wjlj = fj.getCWjlj();
                  if (wjlj != null && !"".equals(wjlj)) {
                      if (StringUtils.isEmpty(ajpcbh)) {
                          File file = new File(wjlj);
                          fj.setNYwmk(YjptConsts.YWMK_SPYJBC_AJPC);
                          fj.setNWjdx(file.length());
                          fjxxService.insertFjByDapc(fj);
                      } else {
                          //编辑时 文件路径为 storage2:ftpStorage....
                          fjxxService.updateTYjptFj(fj);
                      }
                  }
              }
              result.put("success", true);
              transaction.commit(status);
          } catch (Exception e) {
              transaction.rollback(status);
              logger.error("案件评查信息保存失败", e);
              result.put("success", false);
              result.put("msg", "案件评查信息保存失败");
          }

          return result;
      }
  ```


- 问题：代码职责不清晰，Logic层进行了大量的业务处理。

- 改进：

  ```java
  /**
         * 点击时脚本
         * @param item  控件对象
         */
  public Object jqButton432ef_onClickServer(Item item) {
    String zlpcbh = ArteryParamUtil.getString("bh");
    Object ajxinxi = ArteryParamUtil.getString("ajJson");
    JSONArray array = JSONArray.fromObject(ajxinxi);
    zlpcService.saveZlpcList(array, zlpcbh);
    return new BaseResult<String>(true, "添加成功！");
  }
  ```


- 提出人：马梁
- 提出时间：2017-8-8

# Java 面向对象

- 什么是面向对象

  就拿人身体组成来讲的，我们自己有手脚眼口鼻等一系列的器官。来把自己所具有的器官就可以看作我们的属性，自己是不是可以喜怒哀乐和嬉笑怒骂，这些是不是我们的行为，那么自己的具有的属性加自己有的行为就称为一个对象。

  ​	值得注意的是，我们每一个人，一个个的个体就是一个对象，虽然你我各有相同的部分，但是我们也有不一样的，高矮胖瘦等。既然我和你都是人，我们有相同相似的东西，所以我和你同属于人类。人类，就是人的总称，也是相似对象的一种抽象。

  ​	综上所诉，我和你是人类的两个特例，但是外星人也可以用人类称呼我们，看的出来：类的具体表现或者实例就是对象，行话，`New` 一个对象，而对象的抽象或者概括就是类。

- 如何创建对象  

  - 我创建一个“人”对象如下

    ```java
    public class Person {  
    	String name;  
    	int age;  
        String gender;  
        public Person() {  
              
        }  
        Person(String name,int age,String gender){  
            this.name  = name;  
            this.age = age;  
            this.gender = gender;  
            System.out.println(this.name+"对象被创建了"+"，有"+this.age+"岁"+"，			 是"+this.gender+"的");  
        }  
    } 
    ```

  - 创建一个类

    ```java
    public static void main(String[] args) {  
        Person zhangsan = new Person("张三", 18, "男");  
        Person lisi = new Person("李四", 19, "女");  
    } 
    ```

    运行结果如下：

    张三对象被创建了，有18岁，是男的
    李四对象被创建了，有19岁，是女的  

  - 给对象加上“说”行为  

    ```java
    public class Person {  
      String name;  
      int age;  
      String gender;  
      public Person() {  

      }  
      Person(String name,int age,String gender){  
        this.name  = name;  
        this.age = age;  
        this.gender = gender;  
        System.out.println(this.name+"对象被创建了"+"，有"+this.age+"岁"+"，是"+this.gender+"的");  
      }  
      public void say(){  
        System.out.println("我说我叫"+this.name+",别以为我不会说话，我会说很多话。");  
      }  
    ```

  - 类有了方法之后，对象就可以调用这个方法，我们称，此时对象具有类的一些行为表现。

    ```java
    public static void main(String[] args) {  
      Person zhangsan = new Person("张三", 18, "男");  
      zhangsan.say();  
      Person lisi = new Person("李四", 19, "女");  
      lisi.say();  
    }
    ```

    运行结果如下：

    张三对象被创建了，有18岁，是男的
    我说我叫张三,别以为我不会说话，我会说很多话。
    李四对象被创建了，有19岁，是女的
    我说我叫李四,别以为我不会说话，我会说很多话。 

- 综上所述

  ​	类，他有自己的东西，也有给对象的东西。类的东西就是类的成员，类的成员一般有初始化块，构造器，属性，方法，内部类，枚举类。如果是属于类的东西（直接可以用类名.成员调用），则用static调用。其实的东西对象都能用，无论是不是静态的，但是不用static修饰的，就是对象的东西，只能由实例化的对象来调用。

  ​	值得注意的是：**要创建对象，必须调用构造器。 初始化块可以看作是特殊的构造器，无参数传入时调用，创建对象时，反正会被调用。**

  ​	