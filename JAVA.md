# JAVA

## Sting字符串

### String核心特点

* 被final修饰
* 纸条写好字，永远不能涂改；看似修改，实则直接换一张纸条
* 属于引用类型，不是基本类型

### String高频用法（网安刚需）

1.length()：获取纸条字数
2.trim()：擦掉纸条前后空白空格
3.equals()：比对两张纸条文字一不一样
4.contains("a")：纸条里有没有这个字
5.startsWith()：判断开头是不是指定文字
6.endsWith()：判断结尾文字（文件后缀绕过常用）
8.substring(起始下标)：截取纸条一部分，剪切文字
9.split(",")：按照符号把纸条剪成好几张小纸条，返回数组（分割 payload 常用）

```java
#语法
String[] 数组名 = 原字符串.split(",");
```

```java
public class Test {
    public static void main(String[] args) {
        // 一整行文字，用逗号隔开
        String str = "张三,18,男";
        
        // 按逗号切开，装进字符串数组
        String[] arr = str.split(",");
        
        // 打印看结果
        System.out.println(arr[0]); // 张三
        System.out.println(arr[1]); // 18
        System.out.println(arr[2]); // 男
    }
}
```

10.replace("a","b")：把所有 a 换成 b，字符替换绕过
11.toLowerCase()全部小写
12.toUpperCase()全部大写（大小写绕过 waf 专用）

### == 和 equals 致命区别（必考 + 网安核心）

* ==：纸条要一样，数值要一样

* equals：数值一样即可
  ```java
  //1、常量池共用同一张纸条
  String a = "admin";
  String b = "admin";
  System.out.println(a==b); //true 同一张纸条
  System.out.println(a.equals(b));//true 文字一样
  ```

  ```java
  //2、new 单独打印两张纸条
  String a = new String("admin");
  String b = new String("admin");
  System.out.println(a==b); //false 两张不同纸条
  System.out.println(a.equals(b));//true 文字相同
  ```

### StringBuilder & StringBuffer

1.String：印刷死纸条，写好不能改，一改就换新纸条，效率极低
2.StringBuilder：可擦写白板，可以随意涂改、添字，速度飞快，多人同时写会乱（线程不安全)
3.StringBuffer：带锁白板，涂改速度慢，有人写别人不能写（线程安全)

```java
//String：一直新建纸条，很卡
String s="";
for(int i=0;i<1000;i++){
    s+=i;
}

//StringBuilder：一张白板反复写，飞快
StringBuilder sb=new StringBuilder();
for(int i=0;i<1000;i++){
    sb.append(i);
}
```

## 异常

### 两类错误

Error严重故障：摔骨折，救不了（内存爆满、JVM 崩溃）
Exception普通异常：滑倒，可以扶起修复（日常全部异常)

### 编译异常、运行异常

* 编译异常（受检异常）

  ✅比喻：出门前没穿鞋，编译器直接不让出门，写代码就红报错，必须处理

  - 特征：代码没运行就报错，强制必须处理
  - 常见：IO 文件、时间解析、反射部分异常
  - 直白：先天不合格，不准运行

* 运行异常（非受检异常）

  ✅比喻：鞋子穿好了，出门走路踩香蕉皮摔倒，写代码无报错，运行才崩

  - 特征：编译不报错，执行代码才崩溃
  - 常见：算术异常：除以 0            空指针：null.方法()数组下标越界
  - 直白：先天合格，路上出事

* 极简区分

  - 编译异常：没出发直接拦下
  - 运行异常：上路之后摔倒

### try-catch-finally

* 通俗

  * try：大胆走危险路，把容易摔倒的代码放里面
  * catch：摔倒了，接住、补救、不让程序猝死
  * finally：无论摔不摔倒，最后必须做（关门、关灯、释放文件）

* 执行规则

  * try 代码正常：走 try，跳过 catch，执行 finally
  * try 代码报错：立刻终止当前 try，跳进 catch，再执行 finally
  * finally百分百执行

  ```java
  public class Test{
      public static void main(String[] args) {
          try{
              //危险代码：可能摔倒
              int num=1/0;
          }catch(Exception e){
              //摔倒补救：接住异常
              System.out.println("代码出错了");
          }finally{
              //必做操作
              System.out.println("收尾操作");
          }
      }
  }  //攻击者利用异常报错泄露服务器路径；开发者用 catch 隐藏报错。
  ```

### throw、throws（重点极易混淆）

* throws 是甩锅，throw 是当场扔炸弹

* throws：方法头上甩锅

  * 写在方法定义后面

  *  意思：我这个方法有风险，我不处理，谁调用谁自己处理

  * 比喻：店员提前告诉老板：这个东西容易坏，出事你负责

    ```java
    //甩锅：告知调用者有异常
    public static void test() throws Exception{
        int a=1/0;
    }
    ```

* throw：方法内部手动抛异常

  * 写在方法里面
  * 主动制造摔倒，手动抛出异常对象
  * 比喻：直接故意摔地

  ```java
  public static void test(){
      //手动扔异常
      throw new Exception("手动报错");
  }
  ```

* 自定义异常
  * 通俗：官方自带异常不够用，自己造一个专属摔倒方式。
  * 官方只有：滑倒、崴脚。
  * 自定义：低血糖晕倒、中暑晕倒。
  * 写法：自己类继承Exception
  * 用处：精准报错、反序列化构造调用链

```java
//自定义异常：自己造的错误
class AgeException extends Exception{
    //空参构造
    public AgeException(){}
    //有参构造
    public AgeException(String msg){
        super(msg);
    }
}

public class Test{
    public static void checkAge(int age) throws AgeException{
        if(age<0){
            //手动抛出自定义异常
            throw new AgeException("年龄不能负数");
        }
    }

    public static void main(String[] args) {
        try {
            checkAge(-5);
        } catch (AgeException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

* 异常不是意外翻车，大多是明知有风险，只是要不要强制处理的区别。
* e.getMessage()：返回错误原因描述（比如 / by zero 除以 0）

## 常用工具类

### math

* 核心特点
  * 全部静态方法，不用new对象，直接Math.方法()-
  * 负责数学运算、位运算，CTF 数学加密、绕过专用
* 1.abs() 绝对值：负数变正数
* 2.ceil() 向上取整（天花板）
* 3.floor() 向下取整（地板）
* 4.round() 四舍五入
* 5.pow(a,b) 次方，a 的 b 次方
* 6.sqrt() 开平方
* 7.位运算（CTF 重点）&与、|或、^异或、<<左移、>>右移
  * 异或 ^：相同为 0，不同为 1
  * 万能规则：a^b^b=a，CTF 加密解密标配
* 8.Math.random()
  * 生成0.0~1.0随机小数底层自带Random，不是真随机
  * Math.random()底层就是：new Random().nextDouble()

### Random

* 大白核心
  * Java Random 不是真随机，是伪随机！
  * 真随机：扔硬币、天气，无法预判-
  * 伪随机：靠种子 seed + 固定数学公式算出来
  * ✅种子一模一样，所有随机数完全一模一样
* 1.Random()无参
  * 自动拿当前时间戳当种子，每次运行种子不同，随机数不同。
* 2.Random(long seed)有参
  * 手动固定种子，随机序列永久固定，漏洞根源。
  * 常用方法
    * 1.nextInt(n)：生成0～n-1随机整数（最常用）
    * 2.nextDouble()：0.0~1.0 小数
    * 3.nextBoolean()：true/false

* 补充

  * 如果是生成1~8的随机数（比如骰子 1-6），写法是：

    ```java
    // 1~8
    int num = r.nextInt(8) + 1;
    ```

### data

* Date 看不懂年月日，只存一串数字：毫秒时间戳

* 时间戳：1970-01-01 00:00 到现在的总毫秒数。

* 常用方法

  * new Date()：获取当下系统时间

  * getTime()：获取毫秒时间戳（网安高频）

  * new Date(毫秒)：把时间戳转回正常时间

    ```java
    //获取当前时间
    Date d = new Date();
    //获取毫秒时间戳
    long stamp = d.getTime();
    System.out.println(stamp);
    ```

### SimpleDateFormat 时间格式化

* 通俗比喻

  * Date 是纯毫秒乱码，人看不懂
  * 这个类是翻译官，把毫秒翻译成：年-月-日 时:分:秒

* 翻译格式规则

  * yyyy年 MM月 dd日 HH时 mm分 ss秒

* 两个核心功能

  * format(Date)：时间→文字（翻译看懂）

  * parse(字符串)：文字→时间（反向翻译，高危漏洞）

    ```java
    import java.text.SimpleDateFormat;
    import java.util.Date;
    
    public class Test{
        public static void main(String[] args) throws Exception {
            Date d = new Date();
            //规定翻译格式
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            
            //1.时间转文字
            String time = sdf.format(d);
            System.out.println(time);
    
            //2.文字转回时间（编译异常，必须异常处理）
            Date newDate = sdf.parse("2026-05-06 18:00:00");
        }
    }
    ```

* 导包语法
  * java.util：Java 官方的工具文件夹（包）
  * Date：文件夹里的时间工具类（就是咱们学的：存时间戳、获取当前时间）
  * jatava.text：Java 官方的格式化文件夹（包）
  * SimpleDateFormat：文件夹里的时间翻译工具（把时间戳转成 年-月-日）

## IO流

### File类

* 不读文件，只负责找文件，删文件，判断文件

* 高危漏洞：用户可控路径+File=路径穿越（目录遍历）

* 黑客用../跳出当前目录，读取服务器系统密码、配置文件、源码

  ```java
  import java.io.File;
  
  // 网安场景：模拟服务器接收用户传入的文件名，读取文件
  public class FileDemo {
      public static void main(String[] args) {
          // ==============================================
          // 正常情况：用户传入 test.txt
          // ==============================================
          String normalFile = "test.txt";
          File f1 = new File(normalFile);
  
          // 1. 判断文件是否存在（黑客用来探测服务器文件）
          System.out.println("文件是否存在：" + f1.exists());
          // 2. 获取文件绝对路径（泄露服务器真实目录！）
          System.out.println("绝对路径：" + f1.getAbsolutePath());
          // 3. 获取文件名
          System.out.println("文件名：" + f1.getName());
  
          System.out.println("==================================");
  
          // ==============================================
          // 黑客攻击：路径穿越 ../../ 跳出目录
          // 目标：读取 Linux 敏感文件 /etc/passwd
          // Windows：../../Windows/System32/config/SAM
          // ==============================================
          String hackPath = "../../etc/passwd"; // 路径穿越Payload
          File f2 = new File(hackPath);
          System.out.println("黑客路径是否存在：" + f2.exists());
          System.out.println("黑客读取的绝对路径：" + f2.getAbsolutePath());
      }
  }
  ```

*    1.new File(路径)：绑定文件路径，不读取内容
  2. exists()：黑客用来探测服务器有没有敏感文件

  3. getAbsolutePath()：泄露服务器真实目录（代码审计高危点）

  4. ../../：路径穿越，向上跳目录 → 任意文件读取的核心 Payload

  5. isFile():判断是否是文件

  6. isDirectory()：判断是否是文件夹

  7. new File(括号里) 从来不是传 “文件对象”，永远传的是「路径字符串」你说的：
     • 传文件名（比如 test.txt）→ 本质是相对路径
     • 传文件夹路径（比如 D:/java）→ 本质是目录路径
     • 传恶意穿越路径（../../etc/passwd）→ 也是合法路径
     File 类只干一件事：把你给的字符串，包装成一个文件 / 目录路径对象，它不会管这个路径真实存不存在。

  8. getCanonicalPath()：解析后的真实路径

     ```java
     File f2 = new File("../../etc/passwd");
     System.out.println("解析后的真实路径：" + f2.getCanonicalPath());
     //解析后的真实路径：C:\Users\17756\etc\passwd
     ```

    9."."：当前路径

    10.获取当前路径

  ```java
  String currentPath1 = System.getProperty("user.dir");
  System.out.println("【方法1】当前路径：" + currentPath1);
  ```

  ![image-20260507001821170](C:\Users\17756\AppData\Roaming\Typora\typora-user-images\image-20260507001821170.png)

![image-20260507002021009](C:\Users\17756\AppData\Roaming\Typora\typora-user-images\image-20260507002021009.png)

```java
 File f2=new File("../../");
        try{
            System.out.println("获取绝对路径"+f2.getCanonicalPath());
        }catch (IOException e){
            e.printStackTrace();
        }//用这个要import java.io.IOException;
```

* catch (IOException e) = 专门接盘
  ◦ IOException：固定写法，代表输入输出异常（读文件、写文件、解析路径出错都算这个）
  ◦ e：就是报错对象，里面装着「报错原因、报错位置」所有信息
  作用：接住 try 里的错误，不让程序直接挂掉

* e.printStackTrace() = 打印报错详情把错误的原因、在哪一行报错，全部打印到控制台，方便我们找 bug。（这行就是给程序员调试用的，没有实际业务功能

### 字节流、字符流

* 字节流（万能）：读取所有文件（文本、图片、密码、配置、木马）→ 黑客必用

*  字符流（专用）：只读文本 → 辅助使用

*  漏洞：用流读取用户可控路径 → 服务器敏感文件全盘泄露

* db.properties 是什么？

  * 可换成 ../../etc/passwd
  * 直白定义
    这是 Java 项目的「数据库配置文件」，后缀 .properties 是 Java 专用的配置文件格式。里面存的全是顶级敏感信息：
  * 网安核心意义
    黑客攻击的首要目标！拿到这个文件 = 拿到服务器数据库的账号密码 = 可以直接登录数据库删库、盗数据。

* read() 是哪里面的？

  * read() 是 FileInputStream 里面的核心方法！（属于文件字节输入流的自带功能）
  * 作用
    把文件里的二进制数据读到 buffer 水桶里，并返回实际读到了多少个字节。

* String content = new String(buffer, 0, len);

  * Java 字符串的构造方法语法，把「字节数组的一部分」转换成「人类能看懂的字符串
  *    1.buffer：装了文件数据的字节水桶（字节数组）
    2. 0：从水桶的第 0 个位置开始取
    3. len：取实际读到的长度（不取空的部分）

* 在字节流中，导入java.io.IOException干什么？

  *    1.IOException 是 Java 输入 / 输出操作的异常类（IO = Input/Output）
    2. 字节流是操作硬盘文件的代码，天生就会报错，Java 强制要求你必须处理这些错误
    3. 导入 java.io.IOException = 告诉编译器：我要处理文件读写时的所有报错
    4. 网安关键点：处理不好这个异常，会直接泄露服务器信息，变成黑客的突破口！

    ```java
    import java.io.FileInputStream;
    import java.io.IOException;
    
    // 网安场景：黑客读取服务器敏感文件
    public class ByteStreamDemo {
        // 这里声明：我这个方法里会抛出IO异常，让JVM处理
        public static void main(String[] args) throws IOException {
            // ==============================================
            // 攻击：读取敏感文件（模拟：数据库配置文件 db.properties）
            // ==============================================
            String hackFile = "db.properties"; // 可换成 ../../etc/passwd
    
            // 1. 创建字节输入流（硬盘 → 程序：读文件）
            // 👇 这行可能抛异常：文件不存在/权限不足
            FileInputStream fis = new FileInputStream(hackFile);
    
            // 2. 读取文件内容（字节流万能读取）
            byte[] buffer = new byte[1024];
            // 👇 这行可能抛异常：读取中断/文件损坏
            int len = fis.read(buffer); // 读取到字节数组
    
            // 3. 把字节转成字符串（打印文件内容）
            String content = new String(buffer, 0, len);
            System.out.println("===== 读取到敏感文件内容 =====");
            System.out.println(content);
    
            // 4. 关闭流
            // 👇 这行可能抛异常：流关闭失败
            fis.close();
        }
    }
    ```

    ```java
    //优化后的代码
    package test1;
    
    import java.io.FileInputStream;
    import java.io.IOException;
    
    public class ByteStreamDemo{
        public static void main(String[] args) {
            // 1. 先打印当前工作目录，确认程序在找哪里
            System.out.println("当前工作目录：" + System.getProperty("user.dir"));
    
            // 注意：这里的路径要和文件实际位置匹配
            // 比如文件放在项目根目录下，就写 "db.properties"
            // 放在 src 里，要写 "src/db.properties"
            String hackFile = "db.properties";
    
            // 用 try-with-resources 自动关流，同时捕获异常
            // 🔥 流放在 try() 里 → 自动关闭！
            try (FileInputStream fis = new FileInputStream(hackFile)) {
                byte[] buffer = new byte[1024];
                int len = fis.read(buffer);
                String content = new String(buffer, 0, len);
                System.out.println("===== 读取到敏感文件内容 =====");
                System.out.println(content);
            } catch (IOException e) {
                // 安全开发：只给通用提示，不泄露异常详情
                System.out.println("文件读取失败！可能原因：文件不存在/无权限/被占用");
                // 禁止写 e.printStackTrace()！网安大忌，会泄露服务器路径
            }
        }
    }
    ```

* 字符流

  ```java
  package test1;
  
  import java.io.FileReader;
  import java.io.IOException;
  
  public class CharStreamDemo {
      public static void main(String[] args) {
          // 1. 先打印当前工作目录，确认程序在找哪里
          System.out.println("当前工作目录：" + System.getProperty("user.dir"));
          
          // 注意：路径要和文件实际位置匹配
          // 比如文件放在项目根目录下，写 "test.txt"
          // 放在 src/test1 里，写 "src/test1/test.txt"
          String fileName = "test.txt";
  
          // 用 try-with-resources 自动关流，同时捕获异常
          try (FileReader fr = new FileReader(fileName)) {
              char[] buffer = new char[1024];
              int len = fr.read(buffer);
              String content = new String(buffer, 0, len);
              System.out.println("===== 读取文件内容 =====");
              System.out.println(content);
          } catch (IOException e) {
              // 安全开发：只给通用提示，不泄露异常详情（网安大忌！）
              System.out.println("文件读取失败！可能原因：文件不存在/无权限/被占用");
          }
      }
  }
  ```

### 缓冲流

* 缓冲流 = 加速版流（带大水桶，一次读很多）

  * 如果没有缓冲，每读一个字节就和磁盘打一次交道（如同每次只搬一汤匙水，从水源跑到水缸，来回跑）。
    缓冲流在管道中内置了一个大桶（内存缓冲区），它每次从水源舀一大桶水，然后你一小口一小口地从桶里取，直到桶空了再去水源舀。这样大幅减少了系统调用次数。
    写也是一样：数据先写到大桶里，等桶满了或你主动 flush 才一次性倒进磁盘。

* 王牌方法：readLine() → 逐行读取文本

  * readLine() 会一直读直到遇到换行符或回车，所以如果一行无比巨大，就会撑爆内存。
  * readLine() 一次读一行，读到末尾返回 null

* 用途：读取长配置文件、长木马源码，绕过文件过滤检测

* 典型类：BufferedInputStream、BufferedOutputStream、BufferedReader、BufferedWriter。

* java.io.FileReader

  * 专门读取【纯文本文件】的底层流
  * 作用：打开文本文件，和文件建立连接
  *  特点：一个字符一个字符读，速度慢
  *  限制：只能读纯文本（.txt/.properties/.java），不能读图片、视频、exe

* java.io.BufferedReader

  * 给 FileReader 加速的【缓冲包装流】
  * 作用：套在 FileReader 外面，自带缓冲区，一次读一大段数据
  *  王牌功能：readLine() → 一次读一整行（网安读取配置文件必备）
  *  特点：速度极快，不能单独使用，必须依赖 FileReader

* 总：

  * FileReader = 小勺子：一勺一勺舀水（一个字符一个字符读）

  *  BufferedReader = 大水桶：套在勺子外面，一次装一桶水（一次读一大段）

  *  必须配合使用：水桶不能直接舀水，要套在勺子

  * FileReader 是基础：负责和文件对接

  * BufferedReader 是增强：负责提速 + 读整行

    ```java
    import java.io.BufferedReader;
    import java.io.FileReader;
    import java.io.IOException;
    
    public class BufferedReaderDemo {
        public static void main(String[] args) {
            // 用 try-with-resources 自动关闭
            try (BufferedReader br = new BufferedReader(new FileReader("source.txt"))) {
                String line;
                // readLine() 一次读一行，读到末尾返回 null
                while ((line = br.readLine()) != null) {
                    System.out.println(line);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
    ```


### 序列化 —— 把对象冻成“冰雕”，再复活（巨大安全隐患）

* 序列化就是把你程序里的一个活蹦乱跳的对象，连同它的状态冻成一个冰雕（字节序列），你可以把这个冰雕寄送到网络上或存进文件。
  反序列化就是把冰雕重新解冻，复活成一个一模一样的对象。
  ObjectOutputStream 负责冷冻，ObjectInputStream 负责解冻。

* 为什么是渗透测试的重灾区？
  因为解冻（反序列化）的过程中，对象内部可能藏有“定时炸弹”——特殊的 readObject 方法。攻击者可以精心构造一串对象（gadget chain），在解冻的瞬间触发一连串方法调用，最终执行任意系统命令（如反弹 shell），这就是Java 反序列化 RCE 漏洞。

* java.io.* = Java 官方的「文件 / 数据读写工具箱」

  * java.io：Java 专门用来操作文件、读写数据的包（文件夹）
    * 文件夹里装着你刚学的所有工具：File、FileReader、BufferedReader、FileInputStream、ObjectOutputStream（序列化）……
  * *：通配符，意思是把这个包里所有的工具类一次性全部导入

* java.io.Serializable 是一个空的标记接口，里面没有任何方法，作用只有一个：给类贴个标签 → 告诉 JVM：这个类允许做 序列化 / 反序列化

* transient 是 Java 瞬态修饰符被 transient 修饰的成员变量：序列化时直接丢掉、不保存、不网络传输；反序列化后直接给默认值

  * 通俗比喻
    序列化 = 把对象打包成二进制寄快递transient 标记的字段 = 禁止打包带走打包时直接扔掉，收件（反序列化）拿到手，这个字段是空 / 默认值。
  * 核心规则

  1. 只能修饰成员变量，不能修饰类、方法、局部变量
  2. 被 transient 修饰的属性不参与序列化
  3. 反序列化后值恢复为默认值
      ◦ 引用类型：null
      ◦ int：0
      ◦ boolean：false

* public String toString()

  * 一句话核心
    toString() 是 Java 所有对象自带的方法作用：把 对象 转换成 字符串只要你直接 System.out.println(对象)，会自动调用 toString()
  * 不重写会怎么样？
    默认自带的源码逻辑：输出 类名 @哈希地址，完全看不懂，毫无用处。示例输出：
    User@15db9742

* FileOutputStream（文件字节输出流）
  Java 最底层的「文件写字节工具」
  • 作用：只能把「字节数据」写入文件
  • 特点：直接操作硬盘文件，独立使用
  • 限制：不会处理对象，只能写字节

* ObjectOutputStream（对象输出流）
  Java 高级的「对象序列化工具」
  • 作用：把 Java 对象 → 转换成字节数据（序列化）
  • 特点：不能直接写文件，必须依赖底层流
  • 限制：专门配合序列化使用

2. FileInputStream
文件字节输入流作用：从硬盘把文件里的二进制字节读进程序是底层基础流，只负责 raw 字节读取，看不懂什么对象、什么文本，只会搬运字节。
对应之前的 FileOutputStream（往文件写字节），一个读、一个写。

* ObjectInputStream
  对象输入流（反序列化专用）
  • 必须包在 FileInputStream 外面（装饰器模式）
  • 功能：把文件里的二进制数据，还原成 Java 对象
  • 专门用来做反序列化，是 Java 漏洞核心类

* writeObject(对象)
  属于 ObjectOutputStream 的方法    作用：把一个 Java 对象 → 转成二进制 → 写入文件就是序列化的核心动作。

  ```java
  oos.writeObject(person); // 把person对象打包存进文件
  ```

* readObject()
  属于 ObjectInputStream 的方法       作用：把文件里的二进制 → 还原成 Java 对象就是反序列化的核心动作。

* Person p2 = (Person) ois.readObject();

  * readObject() 方法返回值类型是 Object(**public Object readObject()**)，不管你当初存的是 Person、User、Student，它统一返回父类 Object。

  * ois.readObject()
    反序列化读出对象，类型是 Object

  * (Person)
    强制类型转换（向下转型）
    把父类 Object 强行转成子类 Person

  * Person p2
    用 Person 类型变量接收转好的对象

  * Object 是什么？

    * java.lang.Object 是 Java 里所有类的「老祖宗」
    * 是 Java 层级最高的父类，没有比它更顶层的了
    * 所有类：你自己写的 Person、User，系统自带的 String、ArrayList、FileReader天生全是 Object 的子类，不用手动extends Object，Java 自动帮你继承
    * 一句话：Object 是所有 Java 对象的统一祖宗，万物皆 Object

  * 为什么非要设计成「所有类都继承 Object」？

    给所有对象统一标配「通用方法」
    Object 提前写好了一套所有对象都能用的公共方法：
    • toString() 把对象转字符串（你刚学的）
    • equals() 比较两个对象是否相等
    • hashCode() 获取对象哈希地址
    如果没有 Object，每一个类都要自己重新写这些方法，重复、麻烦、混乱。现在：所有类直接继承，拿来就用，还能重写。

  * 统一类型天花板，实现多态
    有了 Object，可以用 Object 类型，接收任意任何类的对象
    Object o1 = new Person();
    Object o2 = new String("abc");
    Object o3 = new FileReader("test.txt");
    不管是什么对象，都能装进 Object 里。
  * 为什么 readObject() 返回值是 Object？
    Person p2 = (Person) ois.readObject();
    • 反序列化时，JVM 不知道当初存的是 Person、User 还是 Student
    • 只能返回最高通用类型 Object
    • 你自己知道原来是 Person，所以要强制向下转型 (Person)
    就是因为 Object 是万物父类，才能统一装下所有对象
  * 网安角度补充
    1. 反序列化漏洞的根基就是 Object 统一父类
        恶意类、正常类都继承 Object，都能被 readObject() 读取、转型、触发代码执行。
    2. 代码审计中：
        任何对象通用接收、类型强转、反序列化，全离不开 Object。

* private static final long serialVersionUID = 1L;

  * 作用：给实现了 Serializable 的类，设置一个固定的序列化

  * 版本号为什么要写：防止你修改类之后，反序列化报错（版本不匹配）

  * 为什么长：Java 语法规定这 4 个修饰符一个都不能少，变量名也必须固定，所以组合起来很长

    ![image-20260507144440837](C:\Users\17756\AppData\Roaming\Typora\typora-user-images\image-20260507144440837.png)

```java
import java.io.*;

// 要被序列化的类必须实现 Serializable 接口（相当于贴上“我可冰冻”的标签）
class Person implements Serializable {
    private static final long serialVersionUID = 1L; // 版本标识
    private String name;
    private transient int age;  // transient 表示“别冰这个”，解冻后为默认值0

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
}

public class SerializationDemo {
    public static void main(String[] args) {
        Person p1 = new Person("Alice", 30);
        String fileName = "person.ser";

        // 1. 冷冻（序列化）
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(fileName))) {
            oos.writeObject(p1);   // 把 p1 冻成冰
            System.out.println("对象已冰冻: " + p1);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 2. 解冻（反序列化）
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(fileName))) {
            Person p2 = (Person) ois.readObject(); // 从冰雕复活，强制转换类型
            System.out.println("解冻后的对象: " + p2);
            // 注意：age 是 transient，输出应该是 0
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

* 补充：类是类类型，和String、int、Person 一样
  * IOException和读写流、文件有关的错比如：文件损坏、路径不对、流打不开、读数据中途中断。就是IO 读写层面出错。
  * ClassNotFoundException
    * 流读出来了二进制数据，但是找不到对应的 Java 类，比如：以前序列化存了 Evil 对象，现在项目里把 Evil 类删了；JVM 拿着二进制数据，不知道这是哪个类，直接报这个异常。
  * 一句话区分
    • IOException：读文件、读流出错
    • ClassNotFoundException：读完了，找不到对应 Java 类
* JVM 是什么？
  * 全称：Java Virtual Machine → Java 虚拟机
  * 一句话：JVM 就是专门用来「运行你写的 Java 代码」的后台隐形程序。
  * 打个最形象的比方：
    • 你写的 Java 代码，是人类能看懂的文字
    • 电脑本身直接看不懂 Java
    • JVM 就是中间翻译官 + 打工小弟
    帮你把 Java 代码翻译成电脑能懂的指令，替你在后台跑程序。

* 唯一固定格式（一字都不能改）

  ```java
  private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
      
      // 1. 先执行默认反序列化
      in.defaultReadObject();
  
      // 2. 这里可以写你自己的代码（正常业务 / 恶意代码）
  }
  ```

  * 必须卡死的 5 条规矩（少一条都不认）
    1. 修饰符必须是 private
    2. 返回值必须是 void
    3. 方法名必须 严格叫 readObject，不能改名
    4. 参数必须是 ObjectInputStream in 类型
    5. 必须抛出 IOException, ClassNotFoundException
        只要你完全照着写 → JVM 识别为钩子

* byte 类型

  * Java 基本数据类型，专门装 -128 ~ 127 之间的整数。

* 
