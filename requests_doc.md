**欢迎关注博主公众号「吾非同」关注Python、测试开发知识分享。**

![](https://pic.wufeitong.com/img/20221115191311.png)

下面是机器+人翻译的requests库官方文档，仅做学习参考，如有不准确之处希望大家指正。


> Requests是一个优雅而简单的Python HTTP库。

**下面，一个请求的案例。**

```
r = requests.get('https://api.github.com/user', auth=('user', 'pass'))
>>> r.status_code
200
>>> r.headers['content-type']
'application/json; charset=utf8'
>>> r.encoding
'utf-8'
>>> r.text
'{"type":"User"...'
>>> r.json()
{'private_gists': 419, 'total_private_repos': 77, ...}
```

你可以非常容易地发送HTTP/1.1请求，不需要在URL中手动添加查询字符串，或者对POST请求表单数据进行编码。并且由于urllib3的存在，HTTP一直是保持长连接的。

### Requests深受欢迎的特点

- 保持长连接

- 国际域名和URL

- 具有Cookie持久性的会话

- 浏览器式SSL验证

- 自动内容解码

- 基本请求/加密验证请求

- 优雅的键/值Cookies

- 自动解压缩

- 统一的响应体

- 支持HTTP(S)代理

- 多部分文件上传

- 流式下载

- 连接超时

- 分块请求

- 支持.netrc


Requests正式支持Python 3.7以上版本，并在PyPy上运行良好。

### Requests安装

使用任何软件包的第一步是正确安装它。

要安装Requests，只需在你选择的终端运行下面这个简单的命令。

```
$ python -m pip install requests
```

### 获取Requests源码

Requests源码在GitHub上积极推动，可以在这里获取源码：https://github.com/psf/requests

你也可以克隆公共仓库。

```
$ git clone git://github.com/psf/requests.git
```

也可以下载压缩包;

```
$ curl -OL https://github.com/psf/requests/tarball/main
# optionally, zipball is also available (for Windows users).
```

一旦你有了源代码的副本，你就可以把它嵌入到你自己的Python包中，或者轻松地把它安装到你的网站包中。

## 快速入门

开始之前，确保已经安装了最新的requests库，具体参考：https://www.wufeitong.com/1080.html

让我们从一个简单的例子开始吧。

### 发出一个请求

首先导入Requests模块。

```
import requests
```

现在，让我们试着获取一个网页。在这个例子中，我们来获取GitHub的公共时间线页面。

```
r = requests.get('https://api.github.com/events')
```

现在，我们有一个叫做r的响应对象，我们可以从这个对象中获得我们需要的所有信息。

Requests的请求API非常简单明显，例如，这是一个POST请求：

```
r = requests.post('https://httpbin.org/post', data={'key': 'value'})
```

很清晰，对吗？那其他的HTTP请求类型呢？PUT、DELETE、HEAD和OPTIONS这些都是类似的。

```
r = requests.put('https://httpbin.org/put', data={'key': 'value'})
>>> r = requests.delete('https://httpbin.org/delete')
>>> r = requests.head('https://httpbin.org/get')
>>> r = requests.options('https://httpbin.org/get')
```

Requests确实非常实用，实现这些这是它所能做的一个开始。

### URL实现参数传递

你经常想在URL的查询字符串中发送一些数据。如果你手动构建URL，这些数据会在URL问号后以键/值对的形式给出，例如httpbin.org/get?key=val。Requests允许你使用params关键字参数，以字符串字典的形式提供这些参数。作为一个例子，如果你想把key1=value1和key2=value2传递给httpbin.org/get，你可以使用下面的代码。

```
payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.get('https://httpbin.org/get', params=payload)
```

你可以打印出URL，可以看出URL已经正确拼接到URL中了：

```
print(r.url)
https://httpbin.org/get?key2=value2&key1=value1
```

注意，任何字典中的键值如果是None，将不会被添加到URL的查询字符串中。

你也可以传递一个列表作为值。

```
payload = {'key1': 'value1', 'key2': ['value2', 'value3']}

>>> r = requests.get('https://httpbin.org/get', params=payload)
>>> print(r.url)
https://httpbin.org/get?key1=value1&key2=value2&key2=value3
```

### 响应内容

我们可以读取服务器的响应内容。再看一下GitHub的时间线页面响应。 

```
import requests

>>> r = requests.get('https://api.github.com/events')
>>> r.text
'[{"repository":{"open_issues":0,"url":"https://github.com/...
```

请求将自动对来自服务器的内容进行解码。大多数unicode字符集是无缝解码的。

当你发出请求时，Requests会根据HTTP头文件对响应的编码进行有根据的猜测。Requests猜测的文本编码在你访问r.text时被使用。你可以使用r.encoding属性找出Requests正在使用的编码，并对其进行修改。

```
r.encoding
'utf-8'
>>> r.encoding = 'ISO-8859-1'
```

如果你改变了编码，当你调用r.text时，Request将使用r.encoding的新值。你可能想在任何情况下这样做，你可以用特殊的逻辑计算出内容的编码。例如，HTML和XML有能力在其请求体中指定其编码。在这样的情况下，你应该使用r.content来找到对应的编码，然后设置r.encoding，这样你的r.text编码才是正确的。

在写特殊场景下，Requests也可以使用自定义编码。如果你已经创建了自己的编码，并在编解码模块中注册了，你可以简单地使用编解码名称作为r.encoding的值，Request将为你处理解码。

### 二进制响应内容

对于非文本请求，你也可以以字节的形式访问响应体。

```
r.content
b'[{"repository":{"open_issues":0,"url":"https://github.com/...
```

 gzip和deflate传输编码会自动为你解密。

如果安装了brotli或brotlicffi库，br传输编码会自动为你解码。

例如，为了从一个请求返回的二进制数据中创建一个图像，你可以使用以下代码。

```
from PIL import Image
>>> from io import BytesIO

>>> i = Image.open(BytesIO(r.content))
```

### JSON响应内容

还有一个内置的JSON解码器，以防你要处理JSON数据。

```
import requests

>>> r = requests.get('https://api.github.com/events')
>>> r.json()
[{'repository': {'open_issues': 0, 'url': 'https://github.com/...
```

如果JSON解码失败，r.json()会抛出一个异常。例如，如果响应得到204（无内容），或者如果响应包含无效的JSON，r.json()会抛出request.exceptions.JSONDecodeError。这种异常可以是不同的python版本和json序列化库可能抛出的异常。

需要注意的是，调用r.json()的成功并不表示响应的成功。一些服务器可能会在失败的响应中返回一个JSON对象（例如，包含HTTP 500的json错误）。这样的JSON将被解码并返回，要检查一个请求是否成功，请使用r.raise_for_status()或检查r.status_code是否是你期望的。

### 原始响应内容

在极特殊的情况下，你想从服务器获得原始的套接字响应，你可以访问r.raw。如果你想这样做，请确保在你的初始请求中设置stream=True。你可以按照如下方式设置请求。

```
r = requests.get('https://api.github.com/events', stream=True)

>>> r.raw
<urllib3.response.HTTPResponse object at 0x101194810>

>>> r.raw.read(10)
'\x1f\x8b\x08\x00\x00\x00\x00\x00\x00\x03'
```

一般来说，你应该使用这样的模式来保存请求到的内容到一个文件中。

```
with open(filename, 'wb') as fd:
    for chunk in r.iter_content(chunk_size=128):
        fd.write(chunk)
```

使用Response.iter_content会处理很多你在直接使用Response.raw时必须处理的问题。当流式下载时，上述方法是检索内容的首选和推荐方法。请注意，chunk_size可以自由调整为一个可能更适合你使用情况的数字。

> 说明：
>
> 关于使用Response.iter_content和Response.raw的一个重要说明。Response.iter_content会自动解码gzip和deflate的传输编码。Response.raw是一个原始的字节流--它不会转换响应内容。如果你真的需要访问它们被返回的字节，请使用Response.raw。

### 自定义请求头

如果你想在请求中添加HTTP头信息，只需在头信息参数中传递一个dict。

例如，我们在前面的例子中没有指定我们的user-agent。

```
url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}

>>> r = requests.get(url, headers=headers)
```

注意：与更具体的信息来源相比，自定义标头的优先级较低。比如说。

- 如果在.netrc中指定了证书，那么用headers=设置的授权头将被覆盖，而这又将被auh=参数覆盖。请求将在~/.netrc、~/_netrc或由NETRC环境变量指定的路径上搜索netrc文件。
- 如果你被重定向到主机外，授权头将被删除。
- 代理授权标头将被URL中提供的代理凭证所覆盖。
- 当我们能够确定内容长度时，Content-Length头文件将被覆盖。

此外，Requests完全不会因为指定了哪些自定义头文件而改变其行为。这些请求头信息只是被传递到最终的请求中。

注意：所有的请求头值必须是字符串、字节码或统一字符。虽然允许，但建议避免传递unicode头值。

### 更加复杂的POST请求

通常情况下，你想发送一些表单数据--比如一个HTML表单。要做到这一点，只需向 data 参数传递一个字典。当请求发出时，你的数据字典将自动进行形式编码。

```
payload = {'key1': 'value1', 'key2': 'value2'}

>>> r = requests.post("https://httpbin.org/post", data=payload)
>>> print(r.text)
{
  ...
  "form": {
    "key2": "value2",
    "key1": "value1"
  },
  ...
}
```

data参数也可以为每个键设置多个值。这可以通过使数据成为一个元组列表或一个以列表为值的字典来实现。当表单有多个元素使用同一个键时，这一点特别有用。

```
payload_tuples = [('key1', 'value1'), ('key1', 'value2')]
>>> r1 = requests.post('https://httpbin.org/post', data=payload_tuples)
>>> payload_dict = {'key1': ['value1', 'value2']}
>>> r2 = requests.post('https://httpbin.org/post', data=payload_dict)
>>> print(r1.text)
{
  ...
  "form": {
    "key1": [
      "value1",
      "value2"
    ]
  },
  ...
}
>>> r1.text == r2.text
True
```

有的时候，你可能想发送没有经过表单编码的数据。如果你传入一个字符串而不是一个dict，这些数据将被直接发布。

例如，GitHub API v3接受JSON编码的POST/PATCH数据。

```
import json

>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, data=json.dumps(payload))
```

请注意，上述代码不会添加Content-Type头（你应该将其设置为application/json）。

如果你需要设置该头信息，而你又不想自己对dict进行编码，你也可以直接使用json参数（在2.4.2版本中加入）传递，它将被自动编码。

```
url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
```

```
r = requests.post(url, json=payload)
```

注意，如果传递的是数据或文件，json参数会被忽略。

### 发送一个多部分编码的文件

Requests使上传多文件变得简单。

```
url = 'https://httpbin.org/post'
>>> files = {'file': open('report.xls', 'rb')}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
```

你可以明确地设置文件名、content_type和请求头。

```
url = 'https://httpbin.org/post'
>>> files = {'file': ('report.xls', open('report.xls', 'rb'), 'application/vnd.ms-excel', {'Expires': '0'})}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "<censored...binary...data>"
  },
  ...
}
```

你也可以发送字符串，作为文件接收。

```
url = 'https://httpbin.org/post'
>>> files = {'file': ('report.csv', 'some,data,to,send\nanother,row,to,send\n')}

>>> r = requests.post(url, files=files)
>>> r.text
{
  ...
  "files": {
    "file": "some,data,to,send\\nanother,row,to,send\\n"
  },
  ...
}
```

如果你要传输一个非常大的文件作为一个multipart/form-data请求，你可能想把请求流化。默认情况下，request不支持这个功能，但有一个单独的包支持这个功能--request-toolbelt。你应该阅读toolbelt的文档以了解更多关于如何使用它的细节。https://toolbelt.readthedocs.io/en/latest/

关于在一个请求中发送多个文件，请参考高级部分（https://requests.readthedocs.io/en/latest/user/advanced/#advanced）。

> 我们强烈建议你以二进制模式打开文件。这是因为Request可能试图为你提供Content-Length头，如果它这样做，这个值将被设置为文件的字节数。如果你以文本模式打开文件，可能会发生错误。

### 响应状态码

我们可以检查响应状态代码。

```
r = requests.get('https://httpbin.org/get')
>>> r.status_code
200
```

请求还带有一个内置的状态代码查询对象，以方便参考。

```
r.status_code == requests.codes.ok
True
```

如果我们做了一个错误的请求（4XX客户端错误或5XX服务器错误），我们可以用Response.raise_for_status()来来抛出来。

```
bad_r = requests.get('https://httpbin.org/status/404')
>>> bad_r.status_code
404

>>> bad_r.raise_for_status()
Traceback (most recent call last):
  File "requests/models.py", line 832, in raise_for_status
    raise http_error
requests.exceptions.HTTPError: 404 Client Error
```

但是，返回状态码是200时，当我们调用raise_for_status()时：

```
r.raise_for_status()
None
```

### 响应请求头

我们可以使用Python字典查看服务器的响应头信息。

```
r.headers
{
    'content-encoding': 'gzip',
    'transfer-encoding': 'chunked',
    'connection': 'close',
    'server': 'nginx/1.0.4',
    'x-runtime': '148ms',
    'etag': '"e1ca502697e5c9317743dc078f67693f"',
    'content-type': 'application/json'
}
```

不过，这个字典很特别：它是专门为HTTP头制作的。根据RFC 7230，HTTP头的名称是不区分大小写的。

因此，我们可以使用任何我们想要的大写字母访问响应头。

```
r.headers['Content-Type']
'application/json'

>>> r.headers.get('content-type')
'application/json'
```

它的特殊之处在于，服务器可以多次发送相同的头和不同的值，但requests 将它们结合起来，这样它们就可以在一个单一的映射中被表示在字典中，正如RFC 7230所规定的（https://www.rfc-editor.org/rfc/rfc7230#section-3.2）。

接收者可以将具有相同字段名的多个标题字段合并为一个 "字段名：字段值 "对，而不改变消息的语义，方法是将每个后续字段值按顺序附加到合并的字段值上，用逗号隔开。

### Cookies

如果你的响应里包含cookies，你可以快速的获取它们。

```
url = 'http://example.com/some/cookie/setting/url'
>>> r = requests.get(url)

>>> r.cookies['example_cookie_name']
'example_cookie_value'
```

将自己的 Cookies发送到服务器，可以使用 cookies 参数:

```
url = 'https://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')

>>> r = requests.get(url, cookies=cookies)
>>> r.text
'{"cookies": {"cookies_are": "working"}}'
```

Cookies在RequestsCookieJar中返回，它的作用类似于dict，但也提供了一个更完整的接口，适合在多个域或路径上使用。Cookie jars也可以被传递到请求中。

```
jar = requests.cookies.RequestsCookieJar()
>>> jar.set('tasty_cookie', 'yum', domain='httpbin.org', path='/cookies')
>>> jar.set('gross_cookie', 'blech', domain='httpbin.org', path='/elsewhere')
>>> url = 'https://httpbin.org/cookies'
>>> r = requests.get(url, cookies=jar)
>>> r.text
'{"cookies": {"tasty_cookie": "yum"}}'
```

### 重定向和历史

默认情况下，Request将对除HEAD以外的所有动词进行位置重定向。

Response.history列表包含了为了完成请求而创建的响应对象。该列表从最早的响应到最近的响应进行排序。

例如，GitHub将所有HTTP请求重定向到HTTPS。

```
r = requests.get('http://github.com/')

>>> r.url
'https://github.com/'

>>> r.status_code
200

>>> r.history
[<Response [301]>]
```

如果你使用GET、OPTIONS、POST、PUT、PATCH或DELETE，你可以用allow_redirects参数禁用重定向处理。

```
r = requests.get('http://github.com/', allow_redirects=False)

>>> r.status_code
301

>>> r.history
[]
```

如果你使用HEAD，你也可以启用重定向功能。

```
r = requests.head('http://github.com/', allow_redirects=True)

>>> r.url
'https://github.com/'

>>> r.history
[<Response [301]>]
```

### 超时

你可以用超时参数告诉Requests在指定的秒数后停止等待响应。几乎所有的生产代码都应该在几乎所有的请求中使用这个参数。如果不这样做，会导致你的程序无限期地挂起。

```
requests.get('https://github.com/', timeout=0.001)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
requests.exceptions.Timeout: HTTPConnectionPool(host='github.com', port=80): Request timed out. (timeout=0.001)
```

> 超时不是对整个响应下载的时间限制；相反，如果服务器在超时秒内没有发出响应（更准确地说，如果在超时秒内没有在底层套接字上收到字节），就会引发异常。如果没有明确指定超时，请求不会超时。

### 错误和异常

如果出现网络问题（如DNS故障、拒绝连接等），Requests将抛出一个ConnectionError异常。

如果HTTP请求返回一个不成功的状态码，Response.raise_for_status()将抛出一个HTTPError异常。

如果一个请求超时，会产生一个超时异常。

如果一个请求超过了配置的最大重定向数，就会产生一个TooManyRedirects异常。

Requests明确引发的所有异常都继承自request.exceptions.RequestException。
