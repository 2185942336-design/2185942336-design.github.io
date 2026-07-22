# HackBar

* HackBar 是一款**浏览器轻量级 Web 安全测试插件**（Firefox/Chrome 均有对应版本），核心作用是**快速修改 HTTP 请求参数、构造攻击 Payload、执行编码解码**，不用离开浏览器就能快速验证 Web 漏洞，是 CTF Web 方向的入门高频工具。
* 本质：是封装了浏览器的请求发送能力，让你不用反复修改地址栏、手写表单，就能快速调整 GET/POST 参数、请求头，测试 SQL 注入、文件包含、SSRF、XSS 等常见漏洞。
* 工具定位：浏览器插件，轻量快速调参工具
* 核心能力：仅支持修改 GET/POST 参数、基础编码、一键生成简单 Payload，只能主动发送构造的请求
* 使用门槛：极低，安装即用，无需额外配置
* 适用场景：CTF 基础题快速验证 Payload、快速调参、简单漏洞测试
* 核心优势：速度快，浏览器内直接操作，不用切换软件
* 简单总结：HackBar 是 “快速调试的小刀”，Burp Suite 是 “全能的工具箱”。简单参数调整、快速试错用 HackBar 效率更高；需要抓包、爆破、重放、构建复杂攻击链则必须用 Burp Suite。

## LOAD

加载当前页面url，点击左上角的 `LOAD` 按钮（快捷键 `Alt+A`），会自动把当前浏览器地址栏的 URL 填充到 HackBar 的输入框，无需手动复制。

## Use POST method

切换请求方式 + 填写参数，如果这道题是 POST 传参，那么打开 Use POST method 开关，下方会出现 POST 参数输入区，按 参数名=参数值 的格式填写，比如某道题的基础测试写法：

```plaintext
url=127.0.0.1
```

如果是 GET 型题目，直接在上方 URL 框里修改参数即可。

## EXECUTE

发送请求，查看结果，点击 `EXECUTE` 按钮（快捷键 `Alt+X`），浏览器页面会直接加载你构造的请求，返回执行结果，无需刷新页面、无需手写表单。

## SPLIT

自动将 URL 中的参数拆分行，方便逐个修改长参数、多参数

## ENCODING

编码解码（URL 编码、Base64、Unicode 等），处理 WAF 绕过、参数编码时高频使用

## HASHING

哈希计算（MD5、SHA 等），遇到需要加密校验的题目可直接使用

## MODIFY HEADER

自定义请求头，可添加 Cookie、User-Agent、Referer 等内容

## SQLi` / `XSS` / `SSRF` / `LFI` / `SSTI

对应漏洞的常用 Payload 快捷菜单，点击即可自动填充到参数中，无需手动输入