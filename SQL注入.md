# SQL注入

* and的优先级高于or
  * 括号 () > NOT > AND > OR

* 3 种 URL 注入专用注释（MySQL
  * ✅ 首选万能注释：--+ （最常用、最稳）
    * 原生 SQL 注释：-- （后面必须带一个空格，少了必报错！）
      URL 中写法：--+
      • 原理：URL 里 + 等价于空格，完美替代必须的空格，浏览器 / 服务器都支持
      • 适用：所有场景，无脑用这个就对了
  * ✅ 次选注释：%23 （MySQL 专属）
    原生 SQL 注释：#
    URL 中写法：%23
    • 原理：# 在 URL 里是页面锚点，浏览器会直接忽略，必须编码成 %23 才生效
    • 适用：MySQL 数据库，和--+效果完全一样

* 1.单引号 ' 和后面的 OR/AND 之间 → 必须加空格
  2. 内容和注释符 --+ 之间 → 必须加空格

* ![image-20260514162840331](C:\Users\17756\AppData\Roaming\Typora\typora-user-images\image-20260514162840331.png)

* xhr类型：它是前后端分离里 “前端偷偷和后台传数据” 的请求类型，和你这个题的/api/接口直接对应，这就是为什么它是真正的注入点。

* MySQL 规则：varchar 字符串字段 和 数字比较，字符串强制转数字（固定规范）

  * 字符串首字符不是数字 → 整体 = 0：'root'→0、'admin'→0
  *  字符串开头是数字 → 截取开头数字：'123abc'→123

* load_file(路径)：读取文件全部源码内容（长字符串）

* if( 正则表达式 , 0 , 1 )

  * 正则为真 → IF 返回数字0 → where username=0 → 页面输出：密码错误
  * 正则为假 → IF 返回数字1 → where username=1 → 页面输出：查询失败

* regexp

  * 等价写法：REGEXP = RLIKE，两个关键词完全通用
    语法：字符串 REGEXP '正则表达式'
    返回：匹配成功 = 1，失败 = 0，原值为 NULL 则返回 NULL
    注意：MySQL 正则是POSIX 标准，没有\d \w \s，不能用 /正则/ 包裹
  * ![image-20260603145033635](C:\Users\17756\AppData\Roaming\Typora\typora-user-images\image-20260603145033635.png)

  * .：匹配任意 1 个字符（不能匹配换行）
    'a1' REGEXP 'a.' →1；'a' REGEXP 'a.'→0

  * []：匹配括号里任意一个字符

    ```sql
    'abc' REGEXP '[abc]' →1
    '5' REGEXP '[0-9]' →1
    'z' REGEXP '[a-z]' →1
    ```

  * [^ ]：取反，匹配不在括号里的字符

    ```sql
    '5' REGEXP '[^a-z]' →1（5 不是小写字母）
    ```

  * 字符串 REGEXP 'ctfshow'：包含匹配（只要里面藏着 ctfshow 就为真）

## 过滤

*  --+ , #, %23
* 空格，/**/，%0b，%09，%0a，%0c，%01，%02，%20
* 关键词小写过滤，大写过滤
* or，||
*  url编码必须是%+两位十六进制，不能多一位，否则直接无效

## 脚本

* GET中的参数是params（requests.get(url,params=xxx)），POST中的参数是data(requests.post(url,data=xxx))

* .format()填空功能

  ```python
  # 模板，留一个空位{}
  template = "今天吃{}"
  # 要填的内容
  food = "米饭"
  # 把food塞进空位
  result = template.format(food)
  # 最后结果：今天吃米饭
  ```

* res.text.find('xxx')= 在网页返回的内容里，搜一下有没有这段文字
  • 搜到 = 猜对了
  • 搜不到（返回 - 1）= 猜错了

* CTF永远用requests发请求，urllib3用来关警告
  * urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
    * 因为我们后面要跳过网站证书验证，Python 会觉得不安全，疯狂弹红色警告文字，这行代码就是告诉 Python：
      别弹烦人的红色提示了！我知道我在跳过证书，不用提醒我！
  * requests.post(url,data=data,verify=False)
    * 关闭 SSL 证书校验
      大白话（核心！解决你之前的报错）：
    * 你访问的是 https 开头的 CTF 网站
    * Python 默认会检查网站的证书（像保安查身份证）
    * CTF 平台的证书是无效的，保安（Python）不让进 → 报错
    * verify=False = 告诉保安：别查身份证了，直接放我进去！

* res.text.find('字符')：查找页面是否有指定字符

* 字符串[:-1] = 取字符串除了「最后一个字符」以外的所有内容

* 在 MySQL 中：TRUE = 1，FALSE = 0MySQL 根本没有真正的「布尔类型」，所谓的 true/false 只是数字 1 和 0 的别名！
  只要做算术运算（+ - * /），MySQL 会自动把：
  • true 转换成 1
  • false 转换成 0

  * char（97个true相加）=a
  * ✅ Python 只负责拼字符串，它根本不知道 MySQL 是什么，也不会计算 true，更不会生成字符 a！
  * ![image-20260525144357642](C:\Users\17756\AppData\Roaming\Typora\typora-user-images\image-20260525144357642.png)

  * char()-->mysql     chr()-->python

* CONCAT()：同一行里，把多个字符串 / 片段拼在一起（普通拼接）   concat(h,e,l,l,o)---->hello
  GROUP_CONCAT()：多行数据，把多行内容合并成一行字符串（聚合拼接）group_concat(name)---->Bob,Cindy,Dannio