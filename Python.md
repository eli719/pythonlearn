# Python

## pip使用国内镜像

### 国内源：

- 清华：https://pypi.tuna.tsinghua.edu.cn/simple
- 阿里云：https://mirrors.aliyun.com/pypi/simple/
- 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
- 华中理工大学：https://pypi.hustunique.com/
- 山东理工大学：https://pypi.sdutlinux.org/ 
- 豆瓣：https://pypi.douban.com/simple/

### 临时使用：

可以在使用pip的时候加参数-i拼接url

例如：pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pyspider，这样就会从清华这边的镜像去安装pyspider库。

### 全局配置：

使用pip命令全局配置pip 阿里云镜像源

```cmd
pip config --global set global.index-url https://mirrors.aliyun.com/pypi/simple/
 
pip config --global set install.trusted-host mirrors.aliyun.com
```





## Requests

Requests是用python语言基于urllib编写的，采用的是Apache2 Licensed开源协议的HTTP库

官方文档：[官方文档](https://requests.readthedocs.io/zh_CN/latest/)，详细的不要不要的

#### 1.基本使用

```python
import requests

response = requests.get("https://www.baidu.com")
print(type(response))
print(response.status_code)
print(response.cookies)
print(type(response.text))
print(response.text)
print(response.content)
print(response.content.decode("utf-8"))
```

结果：

```Terminal
<class 'requests.models.Response'>
200
<class 'str'>
....
```

我们可以看出response使用起来确实非常方便，这里有个问题需要注意一下：
很多情况下的网站如果直接response.text会出现乱码的问题，所以这个使用response.content
这样返回的数据格式其实是二进制格式，然后通过decode()转换为utf-8，这样就解决了通过response.text直接返回显示乱码的问题.

请求发出后，Requests 会基于 HTTP 头部对响应的编码作出有根据的推测。当你访问 response.text 之时，Requests 会使用其推测的文本编码。你可以找出 Requests 使用了什么编码，并且能够使用 response.encoding 属性来改变它.如：

```python
response =requests.get("http://www.baidu.com")
response.encoding="utf-8"
print(response.text)
```

不管是通过response.content.decode("utf-8)的方式还是通过response.encoding="utf-8"的方式都可以避免乱码的问题发生

#### 2.请求

- **带参数的Get请求**

```python
import requests

response = requests.get("http://httpbin.org/get?name=zhaofan&age=23")
print(response.text)
```

如果我们想要在URL查询字符串传递数据，通常我们会通过httpbin.org/get?key=val方式传递。Requests模块允许使用params关键字传递参数，以一个字典来传递这些参数，如果字典中的参数为None则不会添加到url上。

例子如下：

```python
import requests
data = {
    "name":"zhaofan",
    "age":22
}
response = requests.get("http://httpbin.org/get",params=data)
print(response.url)
print(response.text)
```

结果：

```json
{
  "args": {
    "age": "23", 
    "name": "zhaofan"
  }, 
  "headers": {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9", 
    "Accept-Encoding": "gzip, deflate", 
    "Accept-Language": "zh-CN,zh;q=0.9", 
    "Host": "httpbin.org", 
    "Upgrade-Insecure-Requests": "1", 
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36", 
    "X-Amzn-Trace-Id": "Root=1-5f277e1a-3cc5608835d34bd4cc5cf2cc"
  }, 
  "origin": "157.0.142.126", 
  "url": "http://httpbin.org/get?name=zhaofan&age=23"
}
```

**解析json**

```python
import requests
import json

response = requests.get("http://httpbin.org/get")
print(type(response.text))
print(response.json())
print(json.loads(response.text))
print(type(response.json()))
json = response.json()
print(json.get('args'))
```

从结果可以看出requests里面集成的json其实就是执行了json.loads()方法，两者的结果是一样的

**添加headers**

```python
import requests
headers = {
	"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36"
}
response =requests.get("https://www.zhihu.com",headers=headers)

print(response.text)
```

- **基本POST请求**

通过在发送post请求时添加一个data参数，这个data参数可以通过字典构造成，这样对于发送post请求就非常方便，例如：

```python
import requests

data = {
    "name":"zhaofan",
    "age":23
}
response = requests.post("http://httpbin.org/post",data=data)
print(response.text)
```

同样的在发送post请求的时候也可以和发送get请求一样通过headers参数传递一个字典类型的数据

我们可以通过response获得很多属性，例子如下

```python
import requests

response = requests.get("http://www.baidu.com")
print(type(response.status_code),response.status_code)
print(type(response.headers),response.headers)
print(type(response.cookies),response.cookies)
print(type(response.url),response.url)
print(type(response.history),response.history)
```

结果：

```cmd
<class 'int'> 200
<class 'requests.structures.CaseInsensitiveDict'> {'Cache-Control': 'private, no-cache, no-store, proxy-revalidate, no-transform', 'Connection': 'keep-alive', 'Content-Encoding': 'gzip', 'Content-Type': 'text/html', 'Date': 'Mon, 03 Aug 2020 03:16:46 GMT', 'Last-Modified': 'Mon, 23 Jan 2017 13:28:24 GMT', 'Pragma': 'no-cache', 'Server': 'bfe/1.0.8.18', 'Set-Cookie': 'BDORZ=27315; max-age=86400; domain=.baidu.com; path=/', 'Transfer-Encoding': 'chunked'}
<class 'requests.cookies.RequestsCookieJar'> <RequestsCookieJar[<Cookie BDORZ=27315 for .baidu.com/>]>
<class 'str'> http://www.baidu.com/
<class 'list'> []
```

**状态码判断**
Requests还附带了一个内置的状态码查询对象
主要有如下内容：

```pythob
_codes = {

    # Informational.
    100: ('continue',),
    101: ('switching_protocols',),
    102: ('processing',),
    103: ('checkpoint',),
    122: ('uri_too_long', 'request_uri_too_long'),
    200: ('ok', 'okay', 'all_ok', 'all_okay', 'all_good', '\\o/', '✓'),
    201: ('created',),
    202: ('accepted',),
    203: ('non_authoritative_info', 'non_authoritative_information'),
    204: ('no_content',),
    205: ('reset_content', 'reset'),
    206: ('partial_content', 'partial'),
    207: ('multi_status', 'multiple_status', 'multi_stati', 'multiple_stati'),
    208: ('already_reported',),
    226: ('im_used',),

    # Redirection.
    300: ('multiple_choices',),
    301: ('moved_permanently', 'moved', '\\o-'),
    302: ('found',),
    303: ('see_other', 'other'),
    304: ('not_modified',),
    305: ('use_proxy',),
    306: ('switch_proxy',),
    307: ('temporary_redirect', 'temporary_moved', 'temporary'),
    308: ('permanent_redirect',
          'resume_incomplete', 'resume',),  # These 2 to be removed in 3.0

    # Client Error.
    400: ('bad_request', 'bad'),
    401: ('unauthorized',),
    402: ('payment_required', 'payment'),
    403: ('forbidden',),
    404: ('not_found', '-o-'),
    405: ('method_not_allowed', 'not_allowed'),
    406: ('not_acceptable',),
    407: ('proxy_authentication_required', 'proxy_auth', 'proxy_authentication'),
    408: ('request_timeout', 'timeout'),
    409: ('conflict',),
    410: ('gone',),
    411: ('length_required',),
    412: ('precondition_failed', 'precondition'),
    413: ('request_entity_too_large',),
    414: ('request_uri_too_large',),
    415: ('unsupported_media_type', 'unsupported_media', 'media_type'),
    416: ('requested_range_not_satisfiable', 'requested_range', 'range_not_satisfiable'),
    417: ('expectation_failed',),
    418: ('im_a_teapot', 'teapot', 'i_am_a_teapot'),
    421: ('misdirected_request',),
    422: ('unprocessable_entity', 'unprocessable'),
    423: ('locked',),
    424: ('failed_dependency', 'dependency'),
    425: ('unordered_collection', 'unordered'),
    426: ('upgrade_required', 'upgrade'),
    428: ('precondition_required', 'precondition'),
    429: ('too_many_requests', 'too_many'),
    431: ('header_fields_too_large', 'fields_too_large'),
    444: ('no_response', 'none'),
    449: ('retry_with', 'retry'),
    450: ('blocked_by_windows_parental_controls', 'parental_controls'),
    451: ('unavailable_for_legal_reasons', 'legal_reasons'),
    499: ('client_closed_request',),

    # Server Error.
    500: ('internal_server_error', 'server_error', '/o\\', '✗'),
    501: ('not_implemented',),
    502: ('bad_gateway',),
    503: ('service_unavailable', 'unavailable'),
    504: ('gateway_timeout',),
    505: ('http_version_not_supported', 'http_version'),
    506: ('variant_also_negotiates',),
    507: ('insufficient_storage',),
    509: ('bandwidth_limit_exceeded', 'bandwidth'),
    510: ('not_extended',),
    511: ('network_authentication_required', 'network_auth', 'network_authentication'),
}
```



通过下面例子测试：（不过通常还是通过状态码判断更方便）

  ```python
import requests

response= requests.get("http://www.baidu.com")
if response.status_code == requests.codes.ok:
    print("访问成功")
  ```

**文件上传**

实现方法和其他参数类似，也是构造一个字典然后通过files参数传递

```python
import requests
files= {"files":open("git.jpeg","rb")}
response = requests.post("http://httpbin.org/post",files=files)
print(response.text)
```

**获取cookie**

```python
import requests

response = requests.get("http://www.baidu.com")
print(response.cookies)

for key,value in response.cookies.items():
    print(key+"="+value)
```

结果：

```cmd
<RequestsCookieJar[<Cookie BDORZ=27315 for .baidu.com/>]>
BDORZ=27315
```

**会话维持**

cookie的一个作用就是可以用于模拟登陆，做会话维持

```python
import requests
s = requests.Session()
s.get("http://httpbin.org/cookies/set/number/123456")
response = s.get("http://httpbin.org/cookies")
print(response.text)
```

**证书验证**

现在的很多网站都是https的方式访问，所以这个时候就涉及到证书的问题

为了避免这种情况的发生可以通过verify=False
但是这样是可以访问到页面，但是会提示：
InsecureRequestWarning: Unverified HTTPS request is being made. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings InsecureRequestWarning)

```python
import requests
from requests.packages import urllib3
urllib3.disable_warnings()
response = requests.get("https://www.12306.cn",verify=False)
print(response.status_code)
```

这样就不会提示警告信息，当然也可以通过cert参数放入证书路径

  ```python
requests.get('https://github.com', verify='/path/to/certfile')
  ```

**代理设置**

```python
import requests

proxies= {
    "http":"http://127.0.0.1:9999",
    "https":"http://127.0.0.1:8888"
}
response  = requests.get("https://www.baidu.com",proxies=proxies)
print(response.text)
```

若你的代理需要使用HTTP Basic Auth，可以使用 http://user:password@host/ 语法：

```python
proxies = {
    "http": "http://user:pass@10.10.1.10:3128/",
}
```

要为某个特定的连接方式或者主机设置代理，使用 scheme://hostname 作为 key， 它会针对指定的主机和连接方式进行匹配。

```python
proxies = {'http://10.20.1.128': 'http://10.10.1.10:5323'}

```

注意，代理 URL 必须包含连接方式。

如果你的代理是通过sokces这种方式则需要pip install "requests[socks]"

```python
proxies = {
    'http': 'socks5://user:pass@host:port',
    'https': 'socks5://user:pass@host:port'
}
```



**超时设置**

通过timeout参数可以设置超时的时间,

```python
response  = requests.get("https://www.baidu.com",timeout=10)
```

这一 timeout 值将会用作 `connect` 和 `read` 二者的 timeout。如果要分别制定，就传入一个元组：

```python
r = requests.get('https://github.com', timeout=(3.05, 27))

```

如果远端服务器很慢，你可以让 Request 永远等待，传入一个 None 作为 timeout 值，然后就冲咖啡去吧。

```python
r = requests.get('https://github.com', timeout=None)

```

**认证设置**

如果碰到需要认证的网站可以通过requests.auth模块实现

```python
import requests

from requests.auth import HTTPBasicAuth

response = requests.get("http://120.27.34.24:9001/",auth=HTTPBasicAuth("user","123"))
# response = requests.get("http://120.27.34.24:9001/",auth=("user","123"))

print(response.status_code)
```

**异常处理**

关于reqeusts的异常在这里可以看到详细内容：
https://requests.readthedocs.io/zh_CN/latest/api.html#exceptions
所有的异常都是在requests.excepitons中

从源码我们可以看出RequestException继承IOError,
HTTPError，ConnectionError,Timeout继承RequestionException
ProxyError，SSLError继承ConnectionError
ReadTimeout继承Timeout异常

```python
import requests

from requests.exceptions import ReadTimeout,ConnectionError,RequestException


try:
    response = requests.get("http://httpbin.org/get",timeout=0.1)
    print(response.status_code)
except ReadTimeout:
    print("timeout")
except ConnectionError:
    print("connection Error")
except RequestException:
    print("error")
```

## re：正则表达式

#### 1.常用的匹配模式

```python
\w      匹配字母数字及下划线
\W      匹配f非字母数字下划线
\s      匹配任意空白字符，等价于[\t\n\r\f]
\S      匹配任意非空字符
\d      匹配任意数字
\D      匹配任意非数字
\A      匹配字符串开始
\Z      匹配字符串结束，如果存在换行，只匹配换行前的结束字符串
\z      匹配字符串结束
\G      匹配最后匹配完成的位置
\n      匹配一个换行符
\t      匹配一个制表符
^       匹配字符串的开头
$       匹配字符串的末尾
.       匹配任意字符，除了换行符，re.DOTALL标记被指定时，则可以匹配包括换行符的任意字符
[....]  用来表示一组字符，单独列出：[amk]匹配a,m或k
[^...]  不在[]中的字符：[^abc]匹配除了a,b,c之外的字符
*       匹配0个或多个的表达式
+       匹配1个或者多个的表达式
?       匹配0个或1个由前面的正则表达式定义的片段，非贪婪方式
{n}     精确匹配n前面的表达式
{n,m}   匹配n到m次由前面的正则表达式定义片段，贪婪模式
a|b     匹配a或者b
()      匹配括号内的表达式，也表示一个组
```

#### re.match()

尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配的话，match（）就会返回None
语法格式：
re.match(pattern,string,flags=0)

**最常规的匹配**  

```python
import re

content= "hello 123 4567 World_This is a regex Demo"
result = re.match('^hello\s\d\d\d\s\d{4}\s\w{10}.*Demo$',content)
print(result)
print(result.group())  # 获取匹配的结果
print(result.span())  # 获去匹配字符串的长度范围
```

**泛匹配**

```python
import re

content= "hello 123 4567 World_This is a regex Demo"
result = re.match("^hello.*Demo$",content)
print(result)
print(result.group())
print(result.span())
```

**匹配目标**

如果为了匹配字符串中具体的目标，则需要通过（）括起来，例子如下：

```python
import re
content= "hello 1234567 World_This is a regex Demo"
result = re.match('^hello\s(\d+)\sWorld.*Demo$',content)
print(result)
print(result.group())
print(result.group(1))
print(result.span())
```

这里需要说一下的是通过re.group()获得结果后，如果正则表达式中有括号，则re.group(1)获取的就是第一个括号中匹配的结果

**贪婪匹配**

```python
import re

content= "hello 1234567 World_This is a regex Demo"
result= re.match('^hello.*(\d+).*Demo',content)
print(result)
print(result.group(1))
```

从结果中可以看出只匹配到了7，并没有匹配到1234567，出现这种情况的原因是前面的.* 给匹配掉了， .*在这里会尽可能的匹配多的内容，也就是我们所说的贪婪匹配，

如果我们想要匹配到1234567则需要将正则表达式改为：

```python
result= re.match('^he.*?(\d+).*Demo',content)
```

这样结果就可以匹配到1234567

**匹配模式**

很多时候匹配的内容是存在换行的问题的，这个时候的就需要用到匹配模式re.S来匹配换行的内容

```python
import re


content = """hello 123456 world_this
my name is zhaofan
"""

result =re.match('^he.*?(\d+).*?zhaofan$',content,re.S)
print(result)
print(result.group())
print(result.group(1))
```

**转义**

当我们要匹配的内容中存在特殊字符的时候，就需要用到转移符号\,例子如下：

```python
import re

content= "price is $5.00"

result = re.match('price is \$5\.00',content)
print(result)
print(result.group())
```

对上面的一个小结：
尽量使用泛匹配，使用括号得到匹配目标，尽量使用非贪婪模式，有换行符就用re.S
强调re.match是从字符串的起始位置匹配一个模式

#### re.search

re.search扫描整个字符串返回第一个成功匹配的结果

```python
import re

content = "extra things hello 123455 world_this is a Re Extra things"

result = re.search("hello.*?(\d+).*?Re",content)
print(result)
print(result.group())
print(result.group(1))
```

其实这个时候我们就不需要在写^以及$，因为search是扫描整个字符串

注意：所以为了匹配方便，我们会更多的用search，不用match,match必须匹配头部，所以很多时候不是特别方便

#### re.findall

搜索字符串，以列表的形式返回全部能匹配的子串

#### re.sub

替换字符串中每一个匹配的子串后返回替换后的字符串

re.sub(正则表达式，替换成的字符串，原字符串)

```python
import re

content = "Extra things hello 123455 World_this is a regex Demo extra things"

content = re.sub('\d+','',content)
print(content)
```

例子2,在有些情况下我们替换字符的时候，还想获取我们匹配的字符串，然后在后面添加一些内容，可以通过下面方式实现:

```python
import re

content = "Extra things hello 123455 World_this is a regex Demo extra things"

content = re.sub('(\d+)',r'\1 7890',content)
print(content)
```

这里需要注意的一个问题是\1是获取第一个匹配的结果，为了防止转义字符的问题，我们需要在前面加上r

**re.compile**

将正则表达式编译成正则表达式对象，方便复用该正则表达式

```python
import re
content= """hello 12345 world_this
123 fan
"""

pattern =re.compile("hello.*fan",re.S)

result = re.match(pattern,content)
print(result)
print(result.group())
```



