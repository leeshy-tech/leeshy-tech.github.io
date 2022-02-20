# Python实践——后端接口实现(2)


# 增加token验证

书接上文

## token是什么

简单来说，token可以认为是身份令牌，当用户登陆成功之后，获得这个令牌，当需要向服务器请求一些私密资源时，便可以提交这个令牌，以证明自己是合法用户。

## token生成及解析

使用pyjwt这个库：`pip install pyjwt`

### 核心代码：

```python
import jwt
# 加密算法
headers = {
    "alg":"HS256",
    "typ":"JWT"
}
# 密钥
SECRET_KEY = "leeshy"
'''生成一个token'''
def token_encode(user_id) -> str:
    if user_id:
        payload = {
            "user_id":user_id
        }
        token = jwt.encode(payload=payload, key=SECRET_KEY,algorithm='HS256',headers=headers)
        return token
    else:
        return None

'''解码token'''
def token_decode(token) -> str:
    payload = jwt.decode(jwt=token,key=SECRET_KEY,verify=False,algorithms='HS256')
    info = payload["user_id"]
    return info
```

- headers声明加密算法
- SECRET_KEY密钥是编码解码的关键
- payload项中，有一些官方声明项，除官方声明项外还可以存一些自定义信息。![image-20220220202957706](/image/Python/image-20220220202957706.png)

## 用户登陆

如果登陆成功，就返回信息和token，如果不成功，就只返回提示信息。

较上一个版本只有一点改变，核心代码：

```python
'''生成一个返回体'''
def response_body_login(msg,user_id=None):
    response_msg = {
        "msg":msg,
        "token":token_encode(user_id)
    }
    return jsonify(response_msg)

@app.route('/users/login',methods=['POST'])
def users_login():
    if request.method == "POST":
        user_id = request.json.get("user_id")
        user_password = request.json.get("user_password")

        for user_dict in users_list:
            if user_dict["user_id"] == user_id:
                if user_dict["user_password"] == user_password:
                    return response_body_login("success",user_id)
                else: 
                    return response_body_login("password error")

        return response_body_login("id not exist")
```

- 规定的请求体格式是application/json

## 请求用户信息

用户向服务端提供token，验证成功则返回用户头像的url。

```python
@app.route('/users/info',methods=['POST'])
def users_info():
    if request.method == "POST":
        token = request.headers["token"]
        try:
            user_id = token_decode(token)
        except:
            return "token error"

        for user_dict in users_headphotos:
            if user_dict["user_id"] == user_id:
                return user_dict["user_headphoto"]

        return "no headphoto"
```

- token要放在请求头里，所以通过request.headers取得。

## 请求体格式

在前端向后端发送请求时，必须在请求头部分声明请求体格式。

例如：`connection.setRequestProperty("Content-Type","application/json;charset=utf-8");`

这个格式是由后端决定的，否则后端无法取到对应的信息。

### application/x-www-form-urlencoded 

这是最常见的 POST 提交数据的方式，提交时按照键值对`key1=val1&key2=val2`的方式进行编码。

版本1时，就使用了这种方法调试，所以后端代码对应的是：

```python 
user_id = request.form.get("user_id")
user_password = request.form.get("user_password")
```

### application/json

指示服务端消息主体是序列化的JSON字符串。

本次使用的调试方式是json，所以后端代码对应：

```python
user_id = request.json.get("user_id")
user_password = request.json.get("user_password")
```

## 调试

![image-20220220210625365](/image/Python/image-20220220210625365.png)

![image-20220220210720789](/image/Python/image-20220220210720789.png)

##  结束语

### 源码

[https://github.com/leeshy-tech/API_userLogin](https://github.com/leeshy-tech/API_userLogin)

### 参考文献

[【记录】form-data与x-www-form-urlencoded的区别 ](https://www.cnblogs.com/wbl001/p/12050751.html)
