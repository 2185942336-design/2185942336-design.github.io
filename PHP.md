# PHP

## md5

* echo md5("123456")
  * 输出：e10adc3949ba59abbe56e057f20f883e
  * 默认十六进制（false）
* echo md5("123456",true)
  * 输出：乱码（16字节二进制数据）
* false（默认）：返回 32 位十六进制字符串
  * 长度：32字节
* true
  * 返回 原始二进制格式
  * 输出结果：乱码 / 不可读字节流
  * 长度：16字节
* 关键特性 / 漏洞（PHP 独有坑点）
  * 传入数组返回 NULL
    md5([]) 不会报错，直接返回 null（CTF 常用绕过手段）；
  * 弱类型比较漏洞
    MD5 值以 0e 开头的字符串，PHP 会当成数字 0，导致 0 == '0exxxx' 成立；
  * 和其他语言对应
    ◦ Python：hashlib.md5(b"123456").hexdigest()（十六进制）/digest()（二进制）
    ◦ Java：MessageDigest 类的 MD5 加密
    ◦ C：需要调用加密库

## die()

* die() 等价于 python中exit()，是 PHP 最常用的终止脚本函数，作用只有两个：
  * **输出**指定内容；
  * 立即停止执行后续所有代码（后面的代码直接作废）。
  * die("内容")=print("内容"); exit()

## json_encode()

* 把 PHP 数组 / 对象 转换成 JSON 字符串（前端 / 接口通用的数据格式），叫做序列化。

* PHP 关联数组 → JSON 对象（键值对）

  ```php
  $arr = ["msg" => "用户名不存在"];
  echo json_encode($arr); 
  // 输出：{"msg":"用户名不存在"}
  ```

* PHP 索引数组 → JSON 数组

  ```php
  $arr = [1,2,3];
  echo json_encode($arr); 
  // 输出：[1,2,3]
  ```

* 必备参数（解决中文乱码）
  默认会把中文转成 Unicode 编码，加参数 JSON_UNESCAPED_UNICODE 即可正常显示中文：

  ```php
  echo json_encode($arr, JSON_UNESCAPED_UNICODE);
  // 输出：{"msg":"用户名不存在"}（正常中文）
  ```


## preg_metch()

* preg_match(正则规则, 要检查的字符串, [可选：存匹配结果的变量])
  * 匹配到目标内容 → 返回1；
  * 没匹配到 → 返回0；
  *  出错了 → 返回false。

## $ret['msg']

PHP 里关联数组的用法，$ret是一个「带标签的盒子」（数组），'msg'是盒子上的标签，用来存提示信息（比如错误 / 成功提示）。

```php
// 先创建一个空数组（空盒子）
$ret = array(); 
// 给标签'msg'的位置存内容
$ret['键名'] = 值;
//把提示信息存到$ret数组的msg键里，后面用json_encode()把数组打包成 JSON 发给前端，你就能看到页面上的提示了。
```

```php
$ret['msg'] = '登陆成功';
$ret['data'] = []; // 后面存flag用的
echo json_encode($ret); 
// 输出的JSON：{"msg":"登陆成功","data":[]}
```

## is_numric()

* is_numeric(要检查的变量)
  * 是数字 / 数字字符串 → 返回true；
  * 不是数字 → 返回false；
  * !is_numeric就是把结果反过来：true变false，false变true。

## intval()

PHP 里的函数，把变量强制转成整数类型，不管是带小数的数、还是字符串里的数字，都给你抠成整数。

```php
intval(要转换的变量, [可选：进制，默认10])
```

* 转换规则（重点！）
  *  字符串转数字时，只取开头的连续数字部分，非数字开头的字符串直接转成0：
  *  intval("123abc") → 123
  *  intval("abc123") → 0
  *  intval("4a") → 4
  *  intval("root") → 0
  *  小数转整数时，直接去掉小数部分：
  *  intval("123.99") → 123
  *  intval(123.99) → 123

## array_push()

PHP 里的数组函数，给数组追加元素，就像给购物车加东西，直接加到数组末尾。

```php
array_push(目标数组, 要加的元素1, [元素2, ...])
```

目标数组会被直接修改，加上新元素； 返回值是新数组的长度。

```php
array_push($ret['data'], array('flag'=>$flag));
//把包含flag的数组（array('flag'=>$flag)）追加到$ret['data']数组里，这样返回的 JSON 里就会有data字段，里面包含 flag。
```

```php
$ret['data'] = []; // 空数组
$flag = "ctfshow{123456}";
array_push($ret['data'], array('flag'=>$flag));
// 现在 $ret['data'] 变成：[ ['flag'=>'ctfshow{123456}'] ]
```

## array()

盒子：创建数组

* PHP 里用来创建数组的语法，数组就是「能装多个数据的容器」，分两种：
  • 索引数组：按顺序存数据，用数字下标取（比如0、1、2）；
  • 关联数组：给每个数据贴标签（键），用标签取数据（比如'msg'、'flag'）

```php
// 1. 索引数组（简写用[]也可以）
$索引数组 = array('苹果', '香蕉', '橙子');
// 简写：$索引数组 = ['苹果', '香蕉', '橙子'];

// 2. 关联数组（简写用[]也可以）
$关联数组 = array(
    'name' => '张三',
    'age' => 20
);
// 简写：$关联数组 = ['name' => '张三', 'age' => 20];
```

```php
// 关联数组，存用户信息
$user = array(
    'username' => 'root',
    'pass' => 'root',
    'is_admin' => true
);
echo $user['username']; // 输出 root
echo $user['pass']; // 输出 root
```

## PHP代码格式

* <?php    ?>

* <?=      ?>

* <?      ?>

* <script language="php">代码内容</script>

## "."

PHP 里用 . 把两个字符串粘在一起，就像 Python 的 + 或胶水。

```php
"abc" . "123"   // 得到 "abc123"
```

## 伪协议

### php://filter

* PHP 的文件系统就是一个“邮局”。
  你往里存东西（写文件），或者从里拿东西（读文件）。
  php://filter 这个伪协议，是邮局里的一种特殊处理窗口。
  你可以在这个窗口贴一张“处理说明”，要求邮局在存或取的时候对内容动点手脚。
  这个处理说明里就有两条道：
  • write 道：存入（写）的时候走这边
  • read 道：取出（读）的时候走这边

* ✍️ write：存入时处理

  * 你往邮局存一封信，希望信在放进柜子之前被处理一下。
    例子：你写的信是乱码，但你要求邮局先翻译成正常文字再存。

  * ```php
    file_put_contents('php://filter/write=convert.base64-decode/resource=2.php', 'PD9waHAg...');
    ```

  * 过程：
    1. 你把 PD9waHAg... 交给 file_put_contents
    2. 邮局看到你用 write=convert.base64-decode 处理
    3. 在放进 2.php 之前，先对 PD9waHAg... 做 Base64 解码
    4. 解码后的正常文字 <?php @eval(...)?> 被存进 2.php
        关键：处理发生在存入之前，你传入的是 Base64，存进文件的是解码后的 PHP 代码。

* 📖 read：取出时处理

  * 你从邮局取一封信，希望信在交到你手上之前被处理一下。
    例子：柜子里存的是乱码，你要求邮局翻译成正常文字再给你。

  * ```php
    include('php://filter/read=convert.base64-decode/resource=flag.txt');
    ```

  * 过程：
    1. 你要求读取 flag.txt
    2. 邮局看到你用 read=convert.base64-decode 处理
    3. 从 flag.txt 取出内容后，在交给你之前做 Base64 解码
    4. 解码后的内容才到你手上
        关键：处理发生在取出之后、交给你之前。

## $_COOKIE['user']

* $_COOKIE 是 PHP 的超全局变量（一个关联数组），用来获取客户端浏览器发来的 Cookie。

* $_COOKIE['user'] 就是取 Cookie 中名叫 user 的那个值（字符串）。

* 和 $_GET['username'] 类似，只不过一个从 URL 参数取，一个从 Cookie 取。

* 例如，如果你的浏览器 Cookie 里有：

  ```php
  $user = new ctfShowUser();
  ```

  那么 $_COOKIE['user'] 的值就是字符串 "abc123"。

## unserialize(...)

* 美式：/ʌnˈsɪr.i.ə.laɪz/反序列化（计算机术语，指将字符串或字节流恢复为程序中的数据结构或对象）。
* unserialize() 是 PHP 内置函数，作用是把一个序列化的字符串还原成原来的 PHP 值（可以是数组、对象等）。
* 与之相对的是 serialize()，它可以把一个变量变成一个可保存、可传输的字符串。

## serialize()

* PHP 序列化就是把一个变量转换成一种固定格式的字符串，方便存储或传输,你只需要记住下面这几种类型的格式规则，就能看懂甚至手写序列化 payload 了。

### 基本类型格式

* 字符串（String）

  * 格式：s:长度:"内容";

    ```php
    serialize("hello");
    // 结果: s:5:"hello";
    //长度是字节数，不是字符数，所以包含中文时要注意
    ```

* 整数（Integer）

  * 格式：i:数值;

    ```php
    serialize(123);
    // 结果: i:123;
    serialize(-5);
    // 结果: i:-5;
    ```

* 布尔（Boolean）

  * 格式：b:1; 表示 true，b:0; 表示 false

    ```php
    serialize(true);
    // 结果: b:1;
    serialize(false);
    // 结果: b:0;
    ```

* 浮点数（Double / Float）

  * 格式：d:数值;

    ```php
    serialize(3.14);
    // 结果: d:3.14;
    ```

* NULL

  * 格式：N;

    ```php
    serialize(null);
    // 结果: N;
    ```

### 数组

* 格式：a:元素个数:{键;值;键;值;...}

  ```php
  $arr = array("name" => "ctf", "num" => 10);
  serialize($arr);
  // 结果: a:2:{s:4:"name";s:3:"ctf";s:3:"num";i:10;}
  ```

### 对象（Object）

* 格式：O:类名长度:"类名":属性个数:{属性类型 属性名;属性值;...}

  ```php
  class ctfShowUser {
      public $username = 'a';
      public $password = 'a';
      public $isVip = true;
  }
  $obj = new ctfShowUser();
  echo serialize($obj);
  ```

  输出：

  ```txt
  O:11:"ctfShowUser":3:{s:8:"username";s:1:"a";s:8:"password";s:1:"a";s:5:"isVip";b:1;}
  ```


## error_reporting(0)

* 关掉所有错误提示，避免干扰

## highlight_file(______FILE__)

* 把当前PHP代码本身高亮显示在页面上，相当于出题人直接把源码甩你脸上，告诉你「漏洞就在这」

## curl函数

* curl是PHP内置的「网络请求工具」，相当于给服务器装了个全自动跑腿机器人，这四个函数就是这个机器人的操作步骤：
* $ch = curl_init($url);→ 「开机+录入任务」
  * curl_init()：启动这个跑腿机器人，初始化它的工作空间
  * 把$url（就是你POST传的那个url参数）塞给机器人，告诉它：「待会要去访问这个地址」
  * 返回值$ch是这个机器人的「工号」，后面所有操作都要靠这个工号找到它
    👉 漏洞根源就在这：$url完全是你可控的（$_POST['url']），相当于你直接告诉机器人「想去哪就去哪」，没有任何限制。
* curl_setopt($ch, CURLOPT_HEADER, 0);→ 「定规矩1：别带废话」
  * 这是给工号为$ch的机器人定规则：
    •CURLOPT_HEADER是「是否返回HTTP响应头」的开关
    •设为0：意思是「访问目标后，别把HTTP头（比如状态码、Content-Type这些没用的信息）带回来，我只要正文内容」
    •比如访问网页就只拿HTML，访问文件就只拿文件内容，不影响漏洞利用，只是返回结果更干净。
* curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);→ 「定规矩2：把东西存好」
  * 这是最关键的一条规则，直接决定了这是「有回显SSRF」：
    •默认情况下，机器人访问完目标会直接把结果打印到页面上
    •设为1：意思是「别直接打印，把拿到的结果存到一个变量里，等我后面要用」
    •存到哪里？就是后面$result这个变量里。如果没有这条规则，你就算让机器人访问了/etc/passwd，结果也会直接输出，没法被echo控制，但有了这条，你就能拿到完整结果。

* $result = curl_exec($ch);→ 「按下启动键，跑腿开始」
  * 这是真正执行请求的一步（涉及SSRF知识点）：
    •机器人拿着之前录入的$url，严格按照刚才定的两条规矩，真的去访问目标地址
    •访问到的内容全部存到$result变量里
    •这里就是SSRF生效的核心：你传什么scheme它就按什么协议跑：
    •传http://127.0.0.1:8080/admin→ 机器人访问服务器本机的8080端口后台
    •传file:///etc/passwd→ 机器人直接读服务器本地的/etc/passwd文件
    •传gopher://127.0.0.1:6379/_xxx→ 机器人给服务器本机的Redis发TCP指令
    •传http://169.254.169.254/latest/meta-data/→ 机器人去读云服务器的元数据
* curl_close($ch);→ 「下班关机」
  * 机器人跑完任务，把工号$ch注销，释放内存，和漏洞本身没关系，就是清理资源的常规操作。
* echo ($result);→ 「把跑腿结果给你」
  * 把机器人拿回来的内容直接打印到页面上，你作为攻击者就能直接看到服务器访问到的所有内容——这就是标准的「有回显SSRF」。
* 英文单词学习
  * curl= Client URL（客户端URL工具），专门用来让程序像浏览器一样发网络请求的库。
  * init→ initialize（动词）本意：初始化、启动、创建。curl_init()= 初始化curl工具，相当于“开机造出一个跑腿机器人”，同时给它分配一个唯一工号$ch，后面所有操作都靠这个工号找它。
  * setopt→ set option（动词短语，是curl_setopt的口语简写）set= 设置；opt= option（选项）的缩写
    •本意：设置选项/规则
    •代码里：curl_setopt()= 给刚才造出来的机器人定规矩，比如“要不要带响应头”“要不要把结果存起来”，都是在这里设置的。
  * CURLOPT_HEADER→ CURL Option: Header
    •CURL= Client URL（明确是curl库的配置）；OPT= Option（选项）；HEADER= 头部、报文头（HTTP请求/响应里的Content-Type、Status Code这些都属于Header）
    •本意：curl的“是否返回响应头”选项
    •代码里：curl_setopt($ch, CURLOPT_HEADER, 0)= 告诉机器人：“访问完目标后，别把HTTP头这些废话带回来，我只要正文内容”。
  * CURLOPT_RETURNTRANSFER→ CURL Option: Return Transfer
    •RETURN= 返回；TRANSFER= 传输、搬运（指把目标服务器的数据“搬”到当前服务器的过程）
    •本意：curl的“返回传输内容”选项
    •代码里：curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1)= 告诉机器人：“别直接把你拿到的东西扔到页面上，把内容存到变量里，等我后面要用”——这就是SSRF能回显的核心开关。
  * exec→ execute（动词）
    •本意：执行、运行
    •代码里：curl_exec()= 按下机器人的启动键，让它真的去跑腿：按之前设置的规则，访问$url指定的地址，把拿到的结果存到$result里。你传file:///etc/passwd也好，gopher://127.0.0.1:6379也好，都是这一步真的生效。

## parse_url($url)

这是PHP内置的URL解析函数，作用是把一个完整的URL拆成零散的字段，存到数组$x里。
比如你传的$url = "http://127.0.0.1:8080/admin?a=1"，解析后$x的内容是：

```php
$x = [
    'scheme' => 'http',    // 你之前学的协议，这里是http
    'host' => '127.0.0.1', // 主机地址，就是你要打的内网IP
    'port' => 8080,        // 端口，你之前学的8080门
    'path' => '/admin',    // 路径，你之前学的/admin房间
    'query' => 'a=1'       // 参数
];
```

防护作用：把URL拆碎，单独校验每个部分，避免你传畸形URL绕过。

## gethostbyname($x['host'])

* DNS解析主机名为IP
* gethostbyname()是PHP内置的DNS解析函数，作用是把域名（比如baidu.com）转换成真实的IP地址（比如180.101.50.242）。
* 比如你传url=http://sudo.cc/flag.php，$x['host']是sudo.cc，gethostbyname会去查sudo.cc的DNS记录，返回对应的IP。

## filter_var()

* IP合法性+私有IP过滤（核心防护）

  ```php
  if(!filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE)) {
      die('ip!');
  }
  ```

* 这行是内网IP拦截的核心，拆碎了讲：
  •filter_var($ip, FILTER_VALIDATE_IP)：验证$ip是不是合法的IP地址，合法返回IP本身，不合法返回false。
  •FILTER_FLAG_NO_PRIV_RANGE：过滤私有IP段，也就是内网常用的10.0.0.0/8、172.16.0.0/12、192.168.0.0/16，还有你熟悉的127.0.0.0/8（回环地址）。
  •FILTER_FLAG_NO_RES_RANGE：过滤保留IP段，包括云元数据地址169.254.0.0/16、224.0.0.0/4多播地址等。
  •!：逻辑取反，如果filter_var返回false（说明IP是私有/保留IP），!false就是true，执行die('ip!')拦截；如果返回IP本身（公网IP），!IP就是false，放过请求。

* 英文单词：

  * filter_var→ 核心函数
    filter：英文本意是过滤器、筛选器
    var：variable（变量）的缩写
    合起来就是“过滤/校验变量”的函数，专门用来验证一个变量是否符合某种规则，不符合就返回false。
  * FILTER_VALIDATE_IP→ 过滤器类型
    VALIDATE：英文本意是验证、校验（不是“有效”，是“检查是否符合标准”的动作）
    IP：Internet Protocol（互联网协议地址）的缩写
    合起来就是“验证是否为合法IP地址”的过滤器：
    如果$ip是127.0.0.1/192.168.1.1这类合法IP，返回IP字符串本身
    如果$ip是abc/127.0.0这类非法内容，返回false
  * FILTER_FLAG_NO_PRIV_RANGE→ 过滤开关1
    FLAG：英文本意是标志、开关（不是旗子，是“开启/关闭某个规则的开关”）
    NO：否定前缀，意思是“不、禁止”
    PRIV：private（私有的）的缩写（不是privilege“特权”）
    RANGE：英文本意是范围、区间
    合起来就是“禁止私有IP段”的开关，开启后会拦截所有内网私有IP：
    ✅ 拦截范围：你之前学过的10.0.0.0/8、172.16.0.0/12、192.168.0.0/16，还有127.0.0.0/8（回环地址，也就是127.0.0.1所在的段）都属于私有IP，都会被拦。
  * FILTER_FLAG_NO_RES_RANGE→ 过滤开关2
    RES：reserved（保留的）的缩写（不是resource“资源”）
    合起来就是“禁止保留IP段”的开关，开启后会拦截所有IANA预留的、未分配给公网的IP：
    ✅ 拦截范围：你之前学过的云元数据地址169.254.169.254（属于169.254.0.0/16链路本地地址）、多播地址224.0.0.0/4等，都是保留IP，会被拦。

  ## 私有IP和保留IP

* 私有IP（Private IP）—— “内网专用号”
  • 定义：专门用于局域网（比如你家WiFi、公司内部网络）的IP地址，不能在公共互联网（公网）上路由。
  • 三大标准段（记住这三段就够了）：
  ◦ 10.0.0.0 ~ 10.255.255.255（/8，A类）
  ◦ 172.16.0.0 ~ 172.31.255.255（/12，B类）
  ◦ 192.168.0.0 ~ 192.168.255.255（/16，C类）
  • 生活中的例子：你路由器分配给手机、电脑的IP（比如192.168.1.100）就是私有IP。全世界无数个家庭都在用192.168.1.1，但它们互不冲突，因为它们只在各自的内网里有效。

* 保留IP（Reserved IP）—— “特殊用途保留号”
  • 定义：范围更广，指所有未被分配用于公网通信、留作特定技术用途的IP地址。私有IP其实是保留IP中的一个大子集。
  • 除了私有IP，常见的保留IP还包括（这些在CTF里经常考）：
  ◦ 回环地址（Localhost）：127.0.0.0/8（即127.0.0.1~127.255.255.255），代表“本机自己”。
  ◦ 链路本地地址：169.254.0.0/16，当电脑没拿到路由器分配的IP时自动生成的临时地址。
  ◦ 多播地址：224.0.0.0/4，用于一对多的视频广播等。
  ◦ 全零地址：0.0.0.0/8，代表“本主机”或无效地址。

* 关系：保留IP ⊃ 私有IP（保留IP是一个大圈，私有IP是被画在圈内的其中一块）。

## '</br>'

单引号包裹的HTML换行标签
浏览器渲染时会插入一个换行，让内容排版好看

## file_get_contents()

是PHP里最简单粗暴的「一键读内容」函数——你给它一个文件路径或者URL，它就把对应内容一次性全读回来，返回成字符串。
用之前的服务员比喻：之前curl是雇了个专业跑腿机器人，file_get_contents就是服务员自己顺手帮你拿东西，不用复杂配置，但规矩少，很容易出SSRF漏洞。
