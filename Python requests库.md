# Python requests库

## response.text

* 核心含义：response.text 是 requests 库自动帮你把二进制响应内容，转成人类可读字符串的结果。
* 调用逻辑：调用chardet这个编码检测库，猜测这个网页用的编码（比如 UTF-8、GBK），然后用这个编码把二进制内容解码成字符串。
* 类型：str
* 用途：存网页文本、中文内容，可能乱码
* 如果要判断页面有没有 “flag”“查询成功”，用response.text
* 怎么解决response.text的乱码问题？
  * 去网页源码里找<meta charset="xxx">标签，这里写了网页的真实编码
  * 手动指定编码：**response.encoding** = "真实编码"，再用response.text就不会乱码了

## response.content

* 网络上所有数据传输都是二进制bytes类型，而content就是纯二进制文本。response.content 是二进制数据，但展示形式是人类能看懂的简化版，不是原始 01 流。Python 打印 bytes 类型时：自动把二进制翻译成了 ASCII 字符 / 十六进制，方便人类阅读。你看到的 b'<!DOCTYPE html>' = 二进制数据的人类友好展示格式
* response.text = response.content.decode( requests猜出来的编码 )
* 类型：bytes
* 用途：存图片、文件、加密数据，不会乱码
* 如果你要下载靶场的图片，用response.content

## .encode()

* 把字符串转换成字节的形式
  * 字符串无法转换成十六进制，字节才可以

## .hex()

* 转十六进制

## res.status_code

* 获取状态码

## 请求体

* post：res = requests.post(url, **data**=data, verify=False)
* get：params = {"id": "1"}         res = requests.get(url, **params**=params)

## 请求头Headers

* 绕过检测，伪装成正常浏览器访问

* ```python
  headers={
      "User-Agent":"Mozilla/5.0",#伪装浏览器
      "Content-Type":"application/x-www-form-urlencoded"//我发的请求体是「键值对表单」，请按这个规则解析！
  }
  res=requests.post(url,data=data,headers=headers)
  ```

## 超时设置，防止卡死

```python
res=requests.post(url,data=data,timeout=10)
```

