---
date: 2024-09-08
tags:
  - WebCoding
aliases:
  - 从网络中获取数据
---
**需求来源**：衡元空间算法项目目前数据都显示在温度专家（[湿度专家](https://beefindtech.com/ShiDu/haaspc/dist/#/login)）上，其数据目前无法使用Python直接获得，因此使用Python编写一个脚本用来给网页发送请求，以获取网页上的数据

## 所需库
```python
import request
import json
```

## 获取请求信息
发送网络请求需要指定目标URL，可以在对应网页上点击**F12**打开开发者工具，选择**Network**，然后再执行需要执行的动作（比如点击一个按钮），在执行动作后，网页回发送一个请求给服务器，此时就能在Network中看到对应的请求信息：
- Header中可以查看请求头的信息，如**URL**、**Content-Type**（用于告诉接收端接受的数据是以什么格式编码的）、Token等信息
- Payload中可以查看网页发送的请求体的信息（也就是Python请求网页时需要发送的请求的请求体）
- Response中可以查看服务器给接收到的由网页发起的请求的回复，通常包括网页上显示的信息
![[Pasted image 20240908222657.png]]

## 发送对应请求
已知如何获取请求信息之后，就可以使用Python发送对应请求信息的请求了。假如现在要使用账号密码登录[湿度专家](https://beefindtech.com/ShiDu/haaspc/dist/#/login)这个网站，使用网页登录的流程是：
- 进入[湿度专家](https://beefindtech.com/ShiDu/haaspc/dist/#/login)这个网页
- 在账号密码栏输入对应的账号和密码
- 点击登录按钮
  ![[Pasted image 20240908223350.png|800]]
当登录按钮被点击之后，会触发一个提交表单的事件，JavaScript或者其他客户端脚本会从表单中获取用户输入的账号和密码（如果使用的是普通的HTML表单提交，这个步骤会自动完成）**构建成一个请求**，对于登录操作，通常是使用**POST方法**来发送敏感数据（如账号和密码）。请求会包含一个**请求头**和一个**请求体**，请求头包括了**Content-Type**等信息，请求体包含了**表单数据**（即账号和密码）。构建好的请求会被发送给服务器上的特定URL，这个URL通常是处理登录逻辑的服务端接口。服务器接收到请求后，会验证提供的账号和密码是否正确。如果账号密码正确，服务器可能会创建一个会话（session），并生成一个会话ID或**访问令牌（token）**。然后服务器会构建一个HTTP响应，包含一个**状态码**（如200 OK）和可能的响应体（如JSON格式的成功消息、会话ID或token）。服务器将响应返回给客户端。客户端接收到响应后，根据响应的内容更新页面状态。如果登录成功，可能会重定向到主页或其他授权页面，或者更新页面的UI以反映登录状态。

因此，如果要使用Python发送登录的请求，也需要编写请求头和请求体，并且以POST的方式将请求发送给服务器。
首先先使用网页打开登录界面，按F12打开开发者工具，查看Network一栏，在输入账号密码之后，点击“立即登录”，网页在跳转的同时，开发者工具中会捕获网页发送登录请求（通常名为login），在捕获的请求的Headers栏可以查看请求头（Request Headers），可以看到服务器的URL
![[Pasted image 20240908225026.png]]
```python
url = 'https://humigic.com:6983/user/login'
```
可以将请求头中的信息复制下来。在登录这个场景下，请求头只需要包含Content-Type即可，而在某些场景下，可能需要包含Token信息
```python
headers = {
		   'Content-Type': 'application/json;charset=UTF-8'  # 注意'Content-Type'不是下划线，是短横杠
}
```
在开发者工具-Network-login-Payload中，可以看到登录操作需要发送的请求体，可以点击view source全部复制下来粘贴到Python请求体中
![[Pasted image 20240908225620.png]]
```python
Payload =   {
			 "userId":"mkb8ov39",
			 "password":"humi123"
}
```
至此就可以发送POST请求了，使用`requests.post()`方法来发送，方法返回服务器的Response，这里使用一个response变量来承接。具体的response内容和格式可以在开发者工具-Network-login-Response中查看
![[Pasted image 20240908230244.png]]

```python
response = requests.post(url, data=json.dumps(Payload), headers=headers)
```
`requests.post()`方法的`data`参数期望**接收一个字符串或字节流**，因此不能直接传输Python的字典或者列表，**需要使用`json.dumps()`将Python的对象序列化为JSON字符串**
执行上面的语句之后，可以使用`.status_code`来获取状态码，用于指示客户端请求的结果
**常见状态码**如下所示
>**200 OK**: 请求成功，通常用于GET请求。
>**201 Created**: 请求成功并且创建了新的资源，通常用于POST请求。
>**301 Moved Permanently**: 请求的资源已经被永久移动到新的位置。
>**400 Bad Request**: 请求无效或不能被服务器所理解。
>**401 Unauthorized**: 客户端尝试访问需要身份验证的资源但未提供有效的认证凭证。
>**403 Forbidden**: 客户端没有权限访问请求的资源。
>**404 Not Found**: 服务器找不到请求的资源。
>**500 Internal Server Error**: 服务器遇到了一个意外的情况，阻止其完成请求。
>**503 Service Unavailable**: 服务器目前无法处理请求（可能是由于服务器过载或维护）。

可以使用`.text`获取响应的主体内容（body）的字符串
response的类型是<class 'requests.models.Response'>，如果查看状态码发现成功相应，可以使用.json()来将response变成字典便于后续操作
```python
if response.status_code == 200:  
# 成功响应  
	login_response = response.json()  
	token = login_response['data']['token']  # 获取token
	if not token:  
		print("Token not found in response.")  
	else:  
		print('token received')  
  
else:  
	# 错误响应  
	print(f"Login Error: {response.status_code}, {response.text}")
```

下面使用一个**获取温度日志**作为例子演示完整代码：
```python
import requests
import json

url = 'https://humigic.com:6983/web/device/new/infolog'  # 获取日志对应的URL

headers = {
	'Content-Type': 'application/json;charset=UTF-8',
	# 获取温度日志需要在请求头中加入登录页面时获得的token，如果不知道需不需要在请求头中加入token，可以先尝试不加，如果有提示'缺少Token!'的报错再加上
	'token': 'eyJ0eXAiOiJKV1QiLCJhbGciO...'  # token过长不写全
}

payload = {  # 把网页发送的Payload复制过来即可
	"userId":"mkb8ov39",
	"token":"eyJ0eXAiOiJKV1...",  # token过长不写全
	"deivceId":"042FBB60",
	"size":"100",
	"last_row_key":"",
	"startTime":1725781999934,
	"endTime":1725810799934,
	"valueType":"temp"
}

response = requests.post(url, data=json.dumps(payload), headers=headers)

if response.status_code == 200:
	# 成功响应
	res = response.json()
	data = res['data'][-1]  # data是一个由多个字典组成的列表，可以只获取最近的一个字典中的数据
	print(data)
else:
	print(f'Login Error: {response.status_code}, {response.text}')

```
