# Python实践——后端接口实现


# 自己动手写一个API

## 相关概念

### 端口Port

这里端口指的是网络端口，利用IP+端口号可以唯一的定位一台主机上的某个应用程序，可以认为端口是APP之间交换数据的“门”，要想与某APP进行通信，就需要知道它运行在哪个主机的哪个端口上。

我们要写一个API，实质上是写了一个完成特定功能的应用程序，它的接口暴露在外，以供其他程序员调用。

### URL (Uniform Resource Locator)

统一资源定位符，是互联网上标准资源的地址。而互联网上的每个文件都有唯一的一个的URL，它包含的信息指出文件的位置以及浏览器应该怎么处理它。格式为`protocol :// ip[:port] / path / [;parameters][?query]#fragment`

一个应用程序可以暴露多个接口，比如用户登陆、用户注册，url可以是/user/userlogin、/user/usersignup。

### HTTP、HTTPS、POST、GET

超文本传输协议（HTTP）的设计目的是保证客户端与服务器之间的通信。客户端（浏览器）向服务器提交 HTTP 请求；服务器向客户端返回响应。响应包含关于请求的状态信息以及可能被请求的内容。

HTTPS 经由 HTTP 进行通信，但利用 SSL/TLS 来加密数据包，更加安全。

POST和GET都是HTTP协议的请求方法，除了它们还有HEAD、PUT等方法。

- GET方法将查询字符串放在请求的URL中：

  /test/demo_form.php?name1=value1&name2=value2

- POST方法将查询字符串放在HTTP消息主体中：

  ```
  POST /test/demo_form.php HTTP/1.1
  Host: runoob.com
  name1=value1&name2=value2
  ```

总的来说，POST方法更加安全，详细区别可参考结尾文章。

### JSON

JSON: **J**ava**S**cript **O**bject **N**otation(JavaScript 对象表示法)

它采用完全独立于编程语言的文本格式来存储和表示数据。JSON 解析器和 JSON 库支持许多不同的编程语言。 目前非常多的动态（PHP，JSP，.NET）编程语言都支持JSON。

将API看作函数，请求体就是参数，响应体就是返回值，按照JSON格式设计请求体和响应体，前后端程序就能通过JSON解析器互相沟通。

## 准备

### 安装库

```powershell
pip install flask,flask_cors,gevent
```

### 功能设计

设计一个简单的用户登陆功能，前端提供账号密码，服务端返回提示消息。

- 账号密码正确，返回success
- 密码错误，返回password error
- 账号不存在，返回id not exist

### 参数

协议：HTTP（HTTPS会涉及到一些复杂的设置，下次再说。）

方法：POST

请求体示例：

```json
{    
    {"user_id":"2019210777"},
    {"user_password":"123456"}
}
```

响应体示例：

```json
{
    {"msg":"success"}
}
```

## 核心代码

{{< admonition type=info title="关于端口" open=true >}}

程序里的端口号自主选择，不与其他服务冲突即可。如果冲突会报错：地址('0.0.0.0',port)已被使用，被使用的不是IP地址，而是端口号。要么更换端口，要么杀死该端口的服务（Google相关的命令即可）。

 {{< /admonition >}}

```python
from flask import Flask,request,jsonify
from flask_cors import CORS
from gevent import pywsgi

port = 8899
app = Flask(__name__)
CORS(app,resource=r'/*')

users_list = [
    {"user_id":"2019210777","user_password":"123456"},
    {"user_id":"2019210778","user_password":"123456"},
    {"user_id":"2019210779","user_password":"123456"},
    {"user_id":"2019210780","user_password":"123456"},
    {"user_id":"2019210781","user_password":"123456"}
]

'''生成一个返回体'''
def response_body(msg):
    response_msg = [
        {"msg":msg}
    ]
    return jsonify(response_msg)

@app.route('/login',methods=['POST'])
def func():
    if request.method == "POST":
        user_id = request.form.get("user_id")
        user_password = request.form.get("user_password")

        for user_dict in users_list:
            if user_dict["user_id"] == user_id:
                if user_dict["user_password"] == user_password:
                    return response_body("success")
                else: 
                    return response_body("password error")

        return response_body("id not exist")

if __name__ == "__main__":
    server = pywsgi.WSGIServer(('0.0.0.0',port),app)
    server.serve_forever()
    print("end")
```

## 调试

使用Postman进行调试，新建一个请求，填入相关参数。

![image-20220215212008322](/image/Python/image-20220215212008322.png)

![image-20220215211622063](/image/Python/image-20220215211622063.png)

## 部署

### 为什么要部署

写好这个API之后，我尝试写了一个鸿蒙应用来调用，但是始终连接超时，原因如下：

鸿蒙模拟器应当是运行在华为的某台服务器上，它与我的电脑不在同一局域网内，我的电脑的局域网IP，它肯定是访问不到的。

而我用Postman调试，自己访问自己，处于局域网内，所以是没有问题的。

要是想在局域网外进行访问，还是要部署到有公网IP的服务器上。

### 云服务器

我使用的是阿里云服务器

- 防火墙设置

  默认情况下，8899端口的进出流量是不能通过阿里云防火墙的，我们新建一条安全组规则。

  ![image-20220216231932421](/image/Python/image-20220216231932421.png)

  ![image-20220216232024821](/image/Python/image-20220216232024821.png)

- 运行代码

  远程到服务器，执行以下命令：

  安装git：`yum install git`

  下载代码：`git clone https://github.com/leeshy-tech/API_userLogin`

  进入目录：`cd API_userLogin`

  执行代码：`python user_login.py`

- 调试

  在Postman里，url里的IP改为服务器的公网IP，发送请求，调试成功。

  ![image-20220216233908075](/image/Python/image-20220216233908075.png)

  服务器也有相应的输出：

  ![image-20220216232257335](/image/Python/image-20220216232257335.png)

## 结束语

### 项目源码

[https://github.com/leeshy-tech/API_userLogin](https://github.com/leeshy-tech/API_userLogin)

### 参考文献

[HTTP 方法：GET 对比 POST_菜鸟教程](https://www.runoob.com/tags/html-httpmethods.html)

[JSON教程_菜鸟教程](https://www.runoob.com/json/json-tutorial.html)

[https://www.bilibili.com/video/BV1TJ411G7po?spm_id_from=333.999.0.0](https://www.bilibili.com/video/BV1TJ411G7po?spm_id_from=333.999.0.0)

[使用nodejs编写api接口并部署到服务器上](https://www.cnblogs.com/tuspring/p/14340457.html)
