# Java IO  

###  前言

​	Java中IO操作主要是指使Java进行输入，输出操作，Java中所有的操作类都存放在Java.io包中，在使用时需要导入此包。
   在整个Java.io包中最重要的就是5个类和一个接口。5个类指的是`File`、`OutputStream`、`InputStream`、`Writer`、`Reader`；一个接口指的是`Serializable`.掌握了这些IO的核心操作那么对于Java中的IO体系也就有了一个初步的认识了。

### IO 五大类  

- **InputStream**

   InputStream  为字节输入流，它本身为一个抽象类，必须依靠其子类实现各种功能，此抽象类是表示字节输入流的所有类的超类。 继承自InputStream  的流都是向程序中输入数据的，且数据单位为字节（8bit）；下面是InputStream所属的子类：

  ![InputStream](http://353588249-qq-com.iteye.com/upload/picture/pic/73834/68752881-091d-3a59-baf9-2cfdcc19da29.jpg)

  `FileInputStream`：从文件系统中的某个文件中获得的输入字节，FileInputStream 用于读取诸如图像数据之类的原始字节流，示例代码如下：

  ````java
  public class FileCount {   
    publicstatic void main(String[] args) {   
      int count=0;  //统计文件字节长度  
      InputStreamstreamReader = null;   //文件输入流  
      try{  
        streamReader=newFileInputStream(new File("D:/123.jpg"));  
        while(streamReader.read()!=-1) {  //读取文件字节，并递增指针到下一个字节  
          count++;  
        }  
        System.out.println("长度是： "+count+" 字节");  
      }catch (final IOException e) {  
        e.printStackTrace();  
      }finally{  
        try{  
          streamReader.close();  
        }catch (IOException e) {  
          e.printStackTrace();  
        }  
      }  
    } 
  }  
  ````

  如果文件很大，则这样的实现方式容易造成内存溢出，所以引出了缓冲区的概念。可以将`streamReader.read()`改成`streamReader.read(byte[]b)`此方法读取的字节数目等于字节数组的长度，读取的数据被存储在字节数组中，返回读取的字节数，`InputStream`还有其他方法`mark`,`reset`,`markSupported`方法，例如：

  `mark`用于标记当前位置；在读取一定数量的数据(小于`readlimit`的数据)后使用reset可以回到`mark`标记的位置。

  `FileInputStream`不支持`mark/reset`操作；`BufferedInputStream`支持此操作；

  `mark(readlimit)`的含义是在当前位置作一个标记，制定可以重新读取的最大字节数，也就是说你如果标记后读取的字节数大于`readlimit`，你就再也回不到回来的位置了。

  通常`InputStream`的`read()`返回`-1`后，说明到达文件尾，不能再读取。除非使用了`mark/reset`。

- **OutputStream**

  为字节输出流，是整个IO包中字节输出流的最大父类，`OutputStream`类也是一个抽象类，要使用此类必须通过子类实例化对象。
     其子类有：

  ![HashMap数据结构](http://353588249-qq-com.iteye.com/upload/picture/pic/73840/11c14ed2-4843-3e09-b9d6-3ad670977520.jpg)

  ​	`Java I/O`默认是不缓冲流的，所谓“缓冲”就是先把从流中得到的一块字节序列暂存在一个被称为buffer的内部字节数组里，然后你可以一下子取到这一整块的字节数据，没有缓冲的流只能一个字节一个字节读，效率孰高孰低一目了然。有两个特殊的输入流实现了缓冲功能，一个是我们常用的`BufferedInputStream`.示例代码：

  ````java
  public class FileCopy {  
     
    public static void main(String[] args) {  
      //一次取出的字节数大小,缓冲区大小  
       byte[] buffer=new byte[512]; 
       int numberRead=0;  
       FileInputStream input=null;  
       FileOutputStream out =null;  
       try {  
          input=new FileInputStream("D:/123.jpg");
          //如果文件不存在会自动创建  
          out=new FileOutputStream("D:/456.jpg"); 
          //numberRead的目的在于防止最后一次读取的字节小于buffer长度，   
          while ((numberRead=input.read(buffer))!=-1) {  
             out.write(buffer, 0, numberRead);       //否则会自动被填充0  
          }  
       } catch (final IOException e) {
          e.printStackTrace();  
       }finally{  
          try {  
             input.close();  
             out.close();  
          } catch (IOException e) { 
             e.printStackTrace();  
          }  
       }  
    }  
  }  
  ````

  ​

-  **Writer**

  ​	 写入字符流的抽象类。子类必须实现的方法仅有 write(char[], int, int)、flush() 和 close()。但是，多数子类将重写此处定义的一些方法，以提供更高的效率和/或其他功能。 其子类如下：

  ​	![HashMap数据结构](http://353588249-qq-com.iteye.com/upload/picture/pic/73836/13e95630-3067-3b18-9376-8e2c3acbf5c2.jpg)

  `BufferedWriter`：将文本写入字符输出流，缓冲各个字符，从而提供单个字符、数组和字符串的高效写入。

  可以指定缓冲区的大小，或者接受默认的大小。在大多数情况下，默认值就足够大了。

- #### **Reader**

  用于读取字符流的抽象类。子类必须实现的方法只有 read(char[], int, int) 和 close()。但是，多数子类将重写此处定义的一些方法，以提供更高的效率和/或其他功能。 子类有：  

![HashMap数据结构](http://353588249-qq-com.iteye.com/upload/picture/pic/73842/f03703bf-ca4a-34a5-b0dc-8f7782581564.jpg)

- **File**

  java 处理文件的类 File,java提供了十分详细的文件处理方法

  ````java
  public class FileExample{  
    public static void main(String[] args) {  
      createFile();  
    }  

    /** 
     * 文件处理示例 
     */  
    public static void createFile() {  
      File f=new File("d:/test.txt");  
      try{
        //当且仅当不存在具有此抽象路径名指定名称的文件时，不可分地创建一个新的空文件。  
        f.createNewFile(); 
        //查看目录所在磁盘大小（KB）。  
        f.getTotalSpace(); 
        //创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。
        f.mkdirs();
        //删除文件或目录
        f.delete();
        //文件或目录的名称
        f.getName();
        // 返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回 null。  
        f.getParent();
      }catch (Exception e) {  
        e.printStackTrace();  
      }  
    }  
  }  
  ````

  ​

- 总的关系如下

     ![HashMap数据结构](http://img.blog.csdn.net/20140814122633546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTUxMjU5MjE1MQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

  ​