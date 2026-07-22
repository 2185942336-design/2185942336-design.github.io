# 🚩 CTF Web6 解题笔记：源码备份与路径访问

> 针对 **`www.zip`源码泄露** 类题型（Web6）的解题思路与原理分析。

------

## 🔍 1. 题目核心逻辑

此类题目（如 `http://目标网站/www.zip`）通常设计为：

- 

  **提供源码压缩包**：供选手分析网站结构、寻找敏感文件路径。

- 

  **Flag 在服务器端**：真正的 flag 存储在**服务器运行环境**的网站目录下，而非压缩包内。

- 

  **诱导访问**：通过压缩包内的线索（文件名/路径），引导选手直接访问服务器上的对应文件。

------

## 🆚 2. 本地文件 vs 服务器文件（关键区别）

| 来源                        | 文件性质             | 内容说明                                                     |
| --------------------------- | -------------------- | ------------------------------------------------------------ |
| **本地解压的 `fl000g.txt`** | 静态副本（提示文件） | 可能含假 flag 或提示语（如“flag在服务器上”），**非解题答案**。 |
| **URL访问的 `fl000g.txt`**  | 服务器实时文件       | 出题人放置在网站根目录的真 flag 文件，需通过 HTTP 请求获取。 |

------

## 🧩 3. 为什么这样设计？

- 

  **模拟实战**：CTF 考察**远程信息收集与利用**，非本地文件分析。

- 

  **训练思维**：培养“线索在源码，flag 在服务器”的习惯（源码是地图，服务器是宝藏）。

- 

  **防作弊**：避免选手仅解压即获 flag，必须与服务器交互。

------

## ✅ 4. 正确解题步骤（Web6 示例）

1. 

   **下载源码**：访问 `http://题目域名/www.zip`，获取压缩包。

2. 

   **分析线索**：解压后查看文件，发现 `fl000g.txt`的**文件名或路径提示**。

3. 

   **访问真文件**：在浏览器输入 `http://题目域名/fl000g.txt`（直接请求服务器）。

4. 

   **获取 Flag**：页面显示服务器上的真实 flag。

------

## ⚠️ 5. 常见误区

- 

  **错误**：打开本地解压的 `fl000g.txt`，复制内容提交。

- 

  **后果**：获得假 flag 或提示，导致解题失败。

- 

  **原则**：**永远通过 URL 访问服务器文件**获取 flag。

------

## 📝 6. 总结

- 

  **源码压缩包**：仅用于**提供路径线索**（如文件名 `fl000g.txt`）。

- 

  **真实 Flag**：必须通过 **HTTP 请求**（`http://.../fl000g.txt`）获取。

- 

  **核心思维**：本地文件是“地图”，服务器文件是“宝藏”；按图索骥，访问服务器。

# CTF入门信息收集清单

## 一、DNS信息收集

### 1. 常见DNS记录类型及用途

| 记录类型  | 常规用途                | CTF中可能隐藏的信息              |
| --------- | ----------------------- | -------------------------------- |
| A记录     | 域名解析到IPv4地址      | 服务器IP地址，可能指向特定服务器 |
| AAAA记录  | 域名解析到IPv6地址      | IPv6地址信息                     |
| TXT记录   | 文本记录（验证、SPF等） | **最常用**的flag隐藏位置         |
| MX记录    | 邮件服务器地址          | 邮件服务器信息，可能包含flag     |
| CNAME记录 | 域名别名                | 指向其他域名，可能包含线索       |
| NS记录    | 域名服务器记录          | DNS服务器信息                    |
| SOA记录   | 起始授权记录            | 域名管理信息                     |
| PTR记录   | 反向DNS解析             | IP到域名的映射                   |

### 2. 常用查询工具

```
# nslookup（Windows/Linux通用）
nslookup -q=txt target.com        # 查询TXT记录
nslookup -q=mx target.com         # 查询MX记录
nslookup -q=any target.com        # 查询所有记录
nslookup target.com 8.8.8.8       # 指定DNS服务器查询

# dig（Linux/macOS更强大）
dig target.com TXT                # 查询TXT记录
dig target.com ANY                # 查询所有记录
dig @8.8.8.8 target.com TXT       # 指定DNS服务器
dig target.com TXT +short         # 简洁输出

# Windows PowerShell
Resolve-DnsName -Type TXT target.com
```

### 3. 公共DNS服务器

- 

  谷歌DNS：`8.8.8.8`、`8.8.4.4`

- 

  阿里DNS：`223.5.5.5`、`223.6.6.6`

- 

  腾讯DNS：`119.29.29.29`

- 

  114DNS：`114.114.114.114`

- 

  Cloudflare：`1.1.1.1`

## 二、WHOIS信息收集

### 1. 可能包含的信息

- 

  域名注册人姓名、邮箱、电话

- 

  注册商信息

- 

  注册日期、到期日期

- 

  域名服务器信息

- 

  有时会隐藏flag在联系人信息中

### 2. 查询工具

```
# Linux/macOS
whois target.com

# 在线工具
- https://whois.aliyun.com/
- https://whois.cloud.tencent.com/
- https://www.whois.com/
```

## 三、网站文件与目录信息

### 1. 常见隐藏文件

```
robots.txt          # 爬虫规则文件，常包含目录线索
.git/               # Git版本控制目录（可尝试.git泄露）
.svn/               # SVN版本控制目录
.DS_Store           # macOS目录信息文件
index.php.bak       # 备份文件
config.php.bak      # 配置文件备份
www.zip             # 网站源码压缩包
backup/             # 备份目录
admin/              # 管理后台
phpinfo.php         # PHP信息页面
test/               # 测试目录
```

### 2. 目录遍历工具

```
# dirb（经典目录爆破工具）
dirb http://target.com /usr/share/wordlists/dirb/common.txt

# gobuster（更快速）
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt

# ffuf（功能强大）
ffuf -u http://target.com/FUZZ -w wordlist.txt

# 常用字典文件位置
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/seclists/Discovery/Web-Content/common.txt
```

## 四、HTTP响应头信息

### 1. 需要检查的响应头字段

```
Server:              # 服务器类型和版本（Apache/2.4.41等）
X-Powered-By:        # 后端技术（PHP/7.4.3等）
Set-Cookie:          # Cookie信息，可能包含flag
Location:            # 重定向地址，可能包含线索
X-Flag:              # 自定义头部，可能直接放flag
X-Forwarded-For:     # 代理信息
```

### 2. 查看工具

```
# curl查看响应头
curl -I http://target.com
curl -v http://target.com

# 浏览器开发者工具
- Network标签 → 查看请求响应头
- 按F12打开开发者工具
```

## 五、网页源码与注释

### 1. 检查位置

- 

  查看网页源代码（Ctrl+U或右键查看源码）

- 

  检查HTML注释 `<!-- 这里可能藏flag -->`

- 

  JavaScript代码中的注释和字符串

- 

  CSS文件中的注释

- 

  内联JavaScript中的隐藏信息

### 2. 常见套路

- 

  flag直接写在注释中

- 

  注释提示下一步操作

- 

  JS代码中包含加密的flag

- 

  隐藏的表单字段

## 六、图片与文件元数据

### 1. EXIF信息（图片）

- 

  使用`exiftool`查看图片元数据

- 

  flag可能藏在相机型号、GPS坐标、注释字段

```
exiftool image.jpg
```

### 2. 文件类型识别

```
file unknown_file      # 识别文件真实类型
strings file.bin       # 提取文件中的可读字符串
binwalk file.bin       # 分析文件中嵌入的其他文件
```

## 七、子域名枚举

### 1. 常用工具

```
# sublist3r
sublist3r -d target.com

# amass（功能强大）
amass enum -d target.com

# assetfinder
assetfinder --subs-only target.com

# 在线工具
- https://dnsdumpster.com/
- https://crt.sh/（证书透明度日志）
- https://securitytrails.com/
```

### 2. 常见子域名模式

```
admin.target.com
dev.target.com
test.target.com
staging.target.com
api.target.com
mail.target.com
ftp.target.com
```

## 八、端口扫描与服务识别

### 1. 常用工具

```
# nmap基础扫描
nmap -sV -sC target.com    # 版本检测和默认脚本
nmap -p 1-1000 target.com  # 扫描常用端口
nmap -p- target.com        # 扫描所有端口（65535个）

# 常见CTF相关端口
21: FTP（可能允许匿名登录）
22: SSH（可能弱密码）
80: HTTP（Web服务）
443: HTTPS（加密Web服务）
3306: MySQL数据库
6379: Redis数据库
8080: 备用HTTP端口
```

## 九、版本信息泄露

### 1. 常见版本信息位置

- 

  HTTP响应头中的Server、X-Powered-By字段

- 

  网页底部的版权信息

- 

  登录页面的版本提示

- 

  错误页面中的调试信息

- 

  默认安装页面（如phpMyAdmin、WordPress安装）

### 2. 利用方式

- 

  搜索已知版本漏洞

- 

  使用searchsploit查找漏洞利用代码

```
searchsploit apache 2.4.41
```

## 十、其他信息源

### 1. 证书信息

```
# 查看SSL证书信息
openssl s_client -connect target.com:443 -servername target.com
```

### 2. 搜索引擎技巧

```
site:target.com                 # 搜索指定网站内容
inurl:admin site:target.com     # 搜索包含admin的URL
filetype:pdf site:target.com    # 搜索PDF文件
intitle:"index of" site:target.com # 搜索目录列表
```

### 3. 社交媒体与代码仓库

- 

  GitHub搜索公司/项目相关代码

- 

  员工的社交媒体信息

- 

  技术论坛的讨论记录

## 十一、信息收集工作流

### 1. 标准流程

```
1. 阅读题目说明（可能有直接提示）
2. 收集域名信息（DNS、WHOIS）
3. 枚举子域名
4. 端口扫描
5. 网站目录爆破
6. 检查网页源码和响应头
7. 查看robots.txt和常见隐藏文件
8. 检查图片和文件元数据
9. 搜索版本漏洞信息
```

### 2. 工具包推荐

```
# 信息收集工具集合
- nmap：端口扫描和服务识别
- dig/nslookup：DNS查询
- whois：域名信息查询
- dirb/gobuster：目录爆破
- sublist3r/amass：子域名枚举
- exiftool：图片元数据查看
- curl/wget：HTTP请求工具
- searchsploit：漏洞搜索
```

## 十二、注意事项

1. 

   **先读题再操作**：题目说明可能直接给出flag或关键提示

2. 

   **记录所有发现**：使用笔记工具记录所有信息，可能后续需要关联

3. 

   **尝试所有可能性**：如果TXT记录没有，试试MX、CNAME等其他记录

4. 

   **注意大小写和格式**：flag通常有固定格式如`flag{...}`、`CTF{...}`

5. 

   **使用正确工具**：分清工具用途，不要用错工具

6. 

   **保持耐心**：信息收集可能耗时，但这是CTF的基础

## 十三、实战技巧

1. 

   **遇到解析问题时**：更换公共DNS服务器（8.8.8.8、1.1.1.1等）

2. 

   **工具参数记不住时**：使用`--help`查看帮助，或使用交互模式

3. 

   **找不到flag时**：尝试查看网页源代码、检查HTTP响应头、查看robots.txt

4. 

   **多维度验证**：用不同工具验证同一信息，确保准确性

5. 

   **利用在线工具**：当本地工具失效时，使用在线DNS查询、WHOIS查询工具

------

这份清单涵盖了CTF信息收集的主要方面，按照这个框架进行系统化收集，可以避免遗漏关键信息。在实际解题时，可以根据题目类型和提示，有针对性地选择收集方法。记住，信息收集是CTF的基础，扎实的信息收集能力能为后续的漏洞利用打下坚实基础。

# Web目录/路径爆破工具对比与实战指南

------

## 一、四大工具核心定位速查

| 工具          | 开发语言 | 定位               | 性能 | 核心优势                                | 典型适用场景                | 现状                   |
| ------------- | -------- | ------------------ | ---- | --------------------------------------- | --------------------------- | ---------------------- |
| **dirsearch** | Python   | 新手友好型目录扫描 | 中等 | 彩色输出、内置优化、字典丰富            | CTF靶场、轻量测试、新手入门 | 🟢 活跃维护，入门首选   |
| **gobuster**  | Go       | 高性能多模式爆破   | 极高 | 高并发、支持目录/DNS/虚拟主机           | 大规模扫描、真实环境渗透    | 🟢 生产环境主力         |
| **ffuf**      | Go       | 全能HTTP模糊测试器 | 极高 | `FUZZ`占位符、精细过滤、参数/Header爆破 | 复杂CTF、高级渗透、精准过滤 | 🟢 当前CTF主流工具      |
| **dirb**      | Java     | 老牌基础扫描器     | 低   | Kali原生集成、无需安装                  | 传统靶场、基础教学          | 🔴 逐渐淘汰，不建议新学 |

------

## 二、工具深度解析

### 🔹 dirsearch：入门级可视化扫描器

**特点**：

- 

  彩色状态码输出（红/黄/绿），结果一目了然

- 

  内置常用字典（`common.txt`, `extensions.txt`等）

- 

  自动处理Cookies、随机UA、代理等常见需求

- 

  支持递归扫描（`-r`参数）

**基本用法**：

```
python3 dirsearch.py -u https://target.com -e php,html,txt -t 50
# -u 目标URL
# -e 扩展名过滤（逗号分隔）
# -t 线程数（默认20-30即可，过高易被拦截）
```

**适用场景**：快速扫描CTF靶机、寻找常见后台路径（`/admin`, `/login`）、测试个人站点。

------

### 🔹 gobuster：工业级高性能引擎

**核心能力**：

- 

  **目录模式**：`gobuster dir -u ...`（标准目录爆破）

- 

  **DNS模式**：`gobuster dns -d ...`（子域名枚举）

- 

  **虚拟主机模式**：`gobuster vhost -u ...`（同IP多站点识别）

**优势表现**：

```
gobuster dir -u https://target.com -w big_wordlist.txt -t 500 -x php
# -w 指定字典文件（支持超大字典）
# -t 线程数（Go协程并发，可设置数百）
# -x 扩展名附加（如.php/.bak）
```

→ 百万级字典可在几分钟内完成扫描。

**适用场景**：企业渗透测试、大规模资产发现、比赛中的快速子域名枚举。

------

### 🔹 ffuf：CTFer的瑞士军刀

**设计哲学**：**「一切皆可FUZZ」**

- 

  `FUZZ`关键词作为万能占位符，可出现在URL、Header、Body任意位置

- 

  基于响应状态码/长度/内容/正则的精确过滤机制

- 

  支持速率限制、延迟设置、匹配组输出等高级特性

#### ⭐ 解决误报的核心方案

针对**「重定向到默认页导致假阳性」**的问题：

```
# 1. 先获取默认页基准长度
curl -s https://target.com/ | wc -c
# 假设输出 733

# 2. 使用 -fs 过滤该长度（Filter Size）
ffuf -u https://target.com/FUZZ -w wordlist.txt -fs 733
# -fs 排除响应长度为733的结果（即默认页）
# -fc 可排除特定状态码（如 -fc 302,404）
```

#### 高频实战命令集

```
# ① 基础目录爆破（带扩展名）
ffuf -u https://target.com/FUZZ -w common.txt -e .php,.txt,.bak

# ② URL参数爆破（如?id=FUZZ）
ffuf -u "https://target.com/search.php?id=FUZZ" -w params.txt

# ③ POST数据爆破（登录表单）
ffuf -X POST -d "username=FUZZ&password=FUZZ" -u https://target.com/login -w users.txt

# ④ Header头爆破（如API密钥）
ffuf -u https://api.target.com/data -H "Authorization: Bearer FUZZ" -w tokens.txt

# ⑤ 复合过滤（排除长度+包含关键字）
ffuf -u https://target.com/FUZZ -w dict.txt -fs 1024 -fr "error|not found"
# -fr 过滤响应内容的正则表达式
# -mr 仅保留匹配内容的响应
```

**适用场景**：需要绕过WAF过滤、参数模糊测试、API接口探测、精准结果筛选的进阶题目。

------

### 🔹 dirb：传统工具简介

```
dirb https://target.com /usr/share/dirb/wordlists/common.txt
```

**仅建议**在Kali环境且无其他工具时临时使用，或在老旧教程参考场景中使用。

------

## 三、工具选型决策树

```
开始扫描任务
   │
   ├── 场景：CTF靶场/轻量测试 + 需要直观结果？
   │       └─→ 选 dirsearch（易上手，结果可视）
   │
   ├── 场景：真实环境/大字典 + 只需快速发现路径？
   │       └─→ 选 gobuster（性能极致，专注爆破）
   │
   ├── 场景：复杂逻辑（参数/Header/过滤误报/绕过检测）？
   │       └─→ 选 ffuf（灵活强大，解决难题）
   │
   └── 场景：仅有Kali基础环境 + 简单探测？
           └─→ 可选 dirb（备胎方案）
```

------

## 四、CTF实战进阶技巧

### 📌 字典优化策略

- 

  **小字典先行**：先用`common.txt`等精简字典快速试探

- 

  **动态扩展**：根据网站技术栈追加字典（PHP站点加`.php`字典，Java加`.jsp`字典）

- 

  **备份文件探测**：重点关注`.bak`, `.old`, `.swp`, `~`等编辑器备份后缀

### 📌 防封禁措施

```
# ffuf 速率控制示例
ffuf -u https://target.com/FUZZ -w dict.txt -rate 100 -p "0.1-0.3"
# -rate 每秒最大请求数
# -p   每个请求随机延迟范围（秒）
```

### 📌 结果验证原则

- 

  **200≠有效**：务必人工访问确认，防止重定向误导

- 

  **403/401**：可能是权限控制点，尝试`/admin/admin.php`等变形路径

- 

  **500**：可能存在漏洞触发点，值得深入测试

------

## 五、总结与建议

| 学习阶段   | 推荐工具组合                | 核心技能目标                               |
| ---------- | --------------------------- | ------------------------------------------ |
| **入门期** | dirsearch为主               | 掌握状态码含义、基础字典使用、结果初步判断 |
| **进阶期** | ffuf为核心，gobuster为辅    | 掌握FUZZ占位符、精确过滤、参数/Header爆破  |
| **实战期** | ffuf定制化 + gobuster大规模 | 组合使用解决复杂场景，编写自动化扫描脚本   |

> 💡 **立即行动建议**：在你的实验环境中，用ffuf复现之前dirsearch遇到的误报场景，体验`-fs`参数如何一键解决假阳性问题，这是从工具使用者迈向问题解决者的关键一步。

# web12

公开信息泄露：管理员密码的常见套路
很多管理员会用自己的**公开信息**当密码，CTF里这类题的密码来源通常是：
- 题目页面的作者邮箱/ID（比如这题的`h1xa@ctfer.com`）
- 页面源码里的隐藏信息（如注释、meta标签）
- 题目描述里的“无效信息”（比如电话号、昵称）
这题的考点就是让你跳出“必须爆破密码”的思维定式，学会从题目本身找线索。

# web13

本题目的是答题者了解到很多的文章有许多的文档，可以在这些文档中发现很多信息，例如文件中有许多的信息泄露的地方，本题在底部的document这个这个文本中记录到有地址和密码。

# 🚀 浏览器为什么不能直接访问 `/var/www/html/`？

> **一句话结论**：浏览器只能访问**Web服务公开的URL路径**，而无法直接读取服务器本地的**文件系统绝对路径**。Web服务器充当了“翻译官”，将URL映射到对应的本地文件。

------

## 🔍 核心区别：两种路径的本质不同

| 路径类型                        | 例子                                         | 谁在使用               | 说明                                             |
| ------------------------------- | -------------------------------------------- | ---------------------- | ------------------------------------------------ |
| **URL路径 (Web Path)**          | `https://xx.ctf.show/nothinghere/fl000g.txt` | 浏览器、用户           | **对外的大门**。Web服务允许外界访问的地址。      |
| **服务器绝对路径 (Local Path)** | `/var/www/html/nothinghere/fl000g.txt`       | 服务器进程、编辑器后台 | **内部仓库地址**。文件在服务器硬盘上的真实位置。 |

------

## 🏢 形象比喻：大楼与门牌号

想象你要去一家公司取文件：

- 

  **`/var/www/html/`** = 公司的**内部档案库房**（只有员工刷卡才能进）。

- 

  **`/`(网站根目录)** = 公司的**前台接待处**（对外开放）。

- 

  **Web服务器 (Nginx/Apache)** = 前台的**接待员**。

- 

  **浏览器** = 外来的**访客**。

**访问流程如下：**

1. 

   访客（浏览器）说：“我要看 `/nothinghere/fl000g.txt`。”

2. 

   接待员（Web服务器）听到后，转身走进内部库房（`/var/www/html/`），找到了对应的文件。

3. 

   接待员把文件的内容复制一份，拿出来交给访客。

4. 

   **访客永远无法亲自进入库房**，他只能通过接待员间接获取文件内容。

------

## ⚙️ 技术原理解析

在Linux服务器中，Web服务（如Nginx）的配置通常包含类似指令：

```
server {
    root /var/www/html; # 告诉服务器：网站的根目录对应到硬盘的这个位置
    ...
}
```

这意味着：

- 

  当浏览器请求 `/index.html`时，服务器实际是在读取 `/var/www/html/index.html`。

- 

  当你看到编辑器里显示 `/var/www/html/nothinghere/fl000g.txt`时，只需**去掉 `/var/www/html`前缀**，剩下的 `/nothinghere/fl000g.txt`就是浏览器可以识别的合法URL路径。

------

## 💡 CTF实战技巧：路径转换公式

当你通过目录遍历等漏洞看到了服务器的**绝对路径**，想转换成浏览器可访问的**URL**，请记住：

> **浏览器URL = 看到的绝对路径 - `/var/www/html`**

| 场景         | 看到的路径 (编辑器/漏洞回显)           | **浏览器应输入的地址**        |
| ------------ | -------------------------------------- | ----------------------------- |
| 常规情况     | `/var/www/html/admin/config.php`       | `/admin/config.php`           |
| **本题情况** | `/var/www/html/nothinghere/fl000g.txt` | **`/nothinghere/fl000g.txt`** |
| 子目录部署   | `/opt/webapp/public/uploads/shell.jpg` | (需结合上下文推断根目录映射)  |

------

## ❌ 常见误区与避坑

1. 

   **误以为浏览器能读全盘**：试图访问 `/etc/passwd`或 `/var/www/html/...`通常会返回 **403 Forbidden** 或 **404 Not Found**，因为Web服务器被配置为只暴露特定目录，不对外提供整个文件系统的浏览权限。

2. 

   **混淆路径来源**：编辑器有较高权限，所以能“看到”本地路径；但普通Web用户没有这个权限，必须遵守URL规则。

3. 

   **盲目拼接**：直接将编辑器里的绝对路径填到浏览器地址栏会导致访问失败。

**牢记**：你在CTF中发现的任何本地路径，都要先问自己——“它在Web根目录下吗？它的URL等价物是什么？”

# web15

忘记密码也可以尝试点击

# web16

phpinfo()界面开启搜索ctrl+f

探针泄露（tz.php）
\# 探针泄露 & tz.php 大白话整合版 ## 一、探针泄露 探针就好比网站服务器的**全身体检单**，能把服务器版本、文件路径、敏感信息（比如flag）全都展示出来，本来是程序员调试网站用的内部文件，不能给外人看。 泄露就是程序员调试完，**忘了删掉这个体检页面**，导致任何人都能随便访问查看，里面不该暴露的敏感信息被外人看到，这就叫探针泄露。 放到你这道题里，就是网站留了能查看服务器信息的页面，点开后里面直接显示了flag，通过这个页面把敏感信息漏了出去。 **一句话总结**：探针泄露 = 网站没删除调试用的信息页面，导致敏感内容被外人看到。 ## 二、tz.php是什么 1. tz.php是一个PHP网页程序文件，就像.txt是文本文件一样，是网站的程序文件。 2. 它就是这道题里的**探针文件**，文件里写了代码，访问它就会输出服务器的全部体检信息（phpinfo页面）。 3. 文件名里的tz大概率是**探针**的拼音首字母，是随便起的名字，也可以叫info.php、test.php这类。 4. 你访问的`/tz.php?act=phpinfo`，意思就是打开tz.php这个文件，让它执行phpinfo功能，展示服务器信息。 **一句话总结**：tz.php就是这道题里泄露服务器信息和flag的探针文件。

# web18

带 \u 四位码 = Unicode

# web19

后端根本不做解密！它只是直接比较你提交的$p参数值，是否等于代码里硬编码的那个密文字符串。
前端 JS 会自动用 AES 把明文加密

HackBar 就是浏览器里的 “请求快捷修改器”，帮你跳过前端表单的限制，直接和服务器对话，就像 web19 里绕过前端 AES 加密，直接提交服务器认的密文，不用走正常的 “明文→加密” 流程。

加解密工具：[字符 编码/解码 - 锤子在线工具](https://www.toolhelper.cn/EncodeDecode/EncodeDecode)

# web20

dirsearch递归扫描（转进文件夹里扫）：python dirsearch.py -u http://靶机地址 -r 
扫完根目录，自动钻进所有子文件夹继续扫