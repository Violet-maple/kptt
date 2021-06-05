# Python3 requests模块上传文件详解

HTTP 协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须使用什么编码方式。常见的四种编码方式如下： 

1、application/x-www-form-urlencoded 

　　这应该是最常见的 POST 提交数据的方式了。浏览器的原生 form 表单，如果不设置 enctype 属性，那么最终就会以 application/x-www-form-urlencoded 方式提交数据。请求类似于下面这样（无关的请求头在本文中都省略掉了）：

```http
POST http://www.example.com HTTP/1.1    
Content-Type:application/x-www-form-urlencoded;charset=utf-8
title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
```

2、multipart/form-data 

　　除了传统的application/x-www-form-urlencoded表单，我们另一个经常用到的是上传文件用的表单，这种表单的类型为multipart/form-data。
　　这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 form 的 enctyped 等于这个值，下面是示例

接下来我们就来说一下post请求四种传送正文方式：

```http
POST http://www.example.com HTTP/1.1

Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text"
title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png
PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```

3、application/json 
　　application/json 这个 Content-Type 作为响应头大家肯定不陌生。实际上，现在越来越多的人把它作为请求头，用来告诉服务端消息主体是序列化后的 JSON 字符串。由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。　

4、text/xml 
　　它是一种使用 HTTP 作为传输协议，XML 作为编码方式的远程调用规范。

### post请求上传文件四种传送正文方式：

　　（1）请求正文是application/x-www-form-urlencoded

　　（2）请求正文是multipart/form-data

　　（3）请求正文是raw

　　（4）请求正文是binary

（1）请求正文是application/x-www-form-urlencoded

例如：

```python
import requests

url = "http://xxx.xxx/upload"
data = {'key1': 'value1', 'key2': 'value2'}
headers = {'Content-Type': 'application/x-www-form-urlencoded'}
req = requests.post(url, data=data, headers=headers)
print(req.text)
```

Reqeusts支持以form表单形式发送post请求，只需要将请求的参数构造成一个字典，然后传给data参数即可。

输入：

```python
import requests

url = 'http://xxx/xxx/upload'
data = {'key1': 'value1', 'key2': 'value2'}
req = requests.post(url, data=data)
print(req.text)
```

输出：

```python
{ 
“args”: {}, 
“data”: “”, 
“files”: {}, 
“form”: { 
“key1”: “value1”, 
“key2”: “value2” 
}, 
“headers”: { 
…… 
“Content-Type”: “application/x-www-form-urlencoded”, 
…… 
}, 
“json”: null, 
…… 
}
```

可以看到，请求头中的Content-Type字段已设置为application/x-www-form-urlencoded，且`d = {'key1': 'value1', 'key2': 'value2'}`以form表单的形式提交到服务端，服务端返回的form字段即是提交的数据。

（2）请求正文是multipart/form-data

除了传统的application/x-www-form-urlencoded表单，我们另一个经常用到的是上传文件用的表单，这种表单的类型为multipart/form-data。

形式：

```python
import requests


url = "http://xxx/xxx/upload"
data = {'key1': 'value1', 'key2': 'value2'}
headers = {'Content-Type': 'multipart/form-data'}
req = requests.post(url, data=data, headers=headers)
print(req.text)
```

发送文件中的数据需要（安装requests_toolbelt包）

```python
import requests
from requests_toolbelt import MultipartEncoder


mpe = MultipartEncoder(
    fields={
        'field0': 'value', 
        'field1': 'value',
        'field2': ('filename', open('xxx.txt', 'rb'), 'text/plain')
    }
)
url = "http://xxx/xxx/upload"
headers = {'Content-Type': mpe.content_type}
req = requests.post(url, data=mpe, headers=headers)
print(req.text)
```

不需要文件

```python
import requests
from requests_toolbelt import MultipartEncoder


mpe = MultipartEncoder(
	fields={
		'field0': 'value', 
		'field1': 'value'
	}
)
url = "http://xxx/xxx/upload"
headers = {'Content-Type': mpe.content_type}
req = requests.post(url, data=mpe, headers=headers)
print(req.text)
```

（3）请求正文是raw

形式：传入xml格式文本

```python
url = "http://xxx/xxx/upload"
data = '<?xml ?>'
headers = {'Content-Type': 'text/xml'}

req = requests.post(url, data=data, headers=headers)
```

传入json格式文本（可以将一json串传给data参数）

```python
import json
import requests


url = "http://xxx/xxx/upload"
data = json.dumps({'key1': 'value1', 'key2': 'value2'})
headers = {'Content-Type': 'application/json'}

req = requests.post(url, data=data, headers=headers)
print(req.text)
```

或者：

```python
url = "http://xxx/xxx/upload"
params = {'key1': 'value1', 'key2': 'value2'}
headers = {'Content-Type': 'application/json'}

req = requests.post(url, json=params, headers=headers)
print(req.text)
```

输出：

```
{ 
“args”: {}, 
“data”: “{\”key2\”: \”value2\”, \”key1\”: \”value1\”}”, 
“files”: {}, 
“form”: {}, 
“headers”: { 
…… 
“Content-Type”: “application/json”, 
…… 
}, 
“json”: { 
“key1”: “value1”, 
“key2”: “value2” 
}, 
…… 
}
```

（4）请求正文是binary

形式：

```python
import requests


url = "http://xxx/xxx/upload"
files = {'file': open('xxx.xls','rb')}
headers = {'Content-Type': 'binary'}

req = requests.post(url, files=files, headers=headers)
# Requests也支持以multipart形式发送post请求，只需将一文件传给files参数即可。
req = requests.post(url, files=files)

print(req.text)
```

输出：

```
{ 
“args”: {}, 
“data”: “”, 
“files”: { 
“file”: “Hello world!” 
}, 
“form”: {}, 
“headers”: {…… 
“Content-Type”: “multipart/form-data; boundary=467e443f4c3d403c8559e2ebd009bf4a”, 
…… 
}, 
“json”: null, 
…… 
}
```



 **注意：**一定要注意headers/Accept等请求头参数类型。

摘自：[python3 request模块 post请求四种方式](https://blog.csdn.net/whatday/article/details/100165117)

