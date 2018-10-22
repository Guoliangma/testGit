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