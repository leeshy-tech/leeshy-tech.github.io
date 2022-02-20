# 鸿蒙开发实践——用户登陆及个人主页


# 用户登录

## 项目简介

### 项目结构

```
└─ entry
	├─src.main
        ├─ java.com.example.users
            ├─beans
                ├─ Account				API请求体
                └─ LoginMsg				API返回体
            ├─slice
                ├─ UserInfoAbilitySlice		个人主页Slice
                └─ MainAbilitySlice			主页面Slice
            ├─utils
                ├─ DataBaseUtil				数据库操作类
                ├─ HttpRequestUtil			API调用类
                └─ LoadImageUtil			网络图片加载到image组件
            ├─ LocalDBAbility			本地数据库的DataAbility
            ├─ MainAbility
            └─ MyApplication
        └─ resources.base.layout
        	├─ ability_userinfo.xml			个人页面布局文件
            └─ ability_main.xml				主页面布局文件
 	└─ build.gradle							添加外部依赖
```

### 效果

![20220220_220706](/image/HarmonyOS/20220220_220706.gif)

## 项目逻辑

此项目的重点是token，它存储在本地数据库，扮演类似浏览器中cookie的角色。

### 登陆页

- 首页获取用户的账号和密码，点击按钮向后端发送http请求。
- 登陆成功，则将后端返回的token存入本地数据库，并导航到个人页面。
- 登陆不成功，则使用ToastDialog组件显示提示信息，账号错误或密码错误。

### 个人页

个人页面高度精简，只显示用户头像。

- 导航到此页，说明用户一定已经登陆。
- 查询本地数据库，获得token。
- 向后端发送请求，获取用户头像的url。
- 将url加载到image组件。

当用户退出APP，清理后台后，仍然能记住登陆状态，导航回个人主页，不用重新登陆。

## 后端接口

代码：[https://github.com/leeshy-tech/API_userLogin/blob/main/user_login_token.py](https://github.com/leeshy-tech/API_userLogin/blob/main/user_login_token.py)

博客：[Python实践——后端接口实现(2)](https://leeshy-tech.github.io/pythonapi_simplelogin2/)

## 准备

这些都是之前的博客详细说过的部分，不再赘述。

### 添加依赖

`implementation ('com.alibaba:fastjson:1.2.47')`

### 本地数据库的DataAbility——LocalDBAbility

把之前写过的照抄过来，改一下数据库名、表名、建表sql语句即可。

### 请求体和返回体类——Account，LoginMsg

根据后端接口的格式建立的类。

### 网络图片加载——LoadImageUtil

将网络图片加载到image组件，copy之前的项目。

### API调用类——HttpRequestUtil

发送HTTP请求的封装类，这个之前没讲过，照抄会用即可。

## DataBaseUtil

为了尽可能的简化主体代码，把从数据库里查询token，和插入token的操作封装成这个类。

```Java
//数据库查询、插入 帮助类
public class DataBaseUtil {
    private static Uri uri = Uri.parse("dataability:///com.example.users.LocalDBAbility/user_info");
    public static String getToken(Context context){
        String value = null;

        DataAbilityHelper dataAbilityHelper = DataAbilityHelper.creator(context);
        String[] colums = {"token"};
        DataAbilityPredicates predicates = new DataAbilityPredicates();
        try {
            ResultSet rs = dataAbilityHelper.query(uri, colums, predicates);
            if(rs.getRowCount() >0){
                rs.goToFirstRow();
                value = rs.getString(0);
            }
        } catch (DataAbilityRemoteException e) {
            e.printStackTrace();
        }
        return value;
    }
    public static int setToken(String token,Context context) {
        int i = 0;
        ValuesBucket valuesBucket = new ValuesBucket();
        valuesBucket.putString("token",token);
        DataAbilityHelper dataAbilityHelper = DataAbilityHelper.creator(context);
        try {
            i = dataAbilityHelper.insert(uri, valuesBucket);
        } catch (DataAbilityRemoteException e) {
            e.printStackTrace();
        }
        return i;
    }
}
```

## MainAbilitySlice

onStart方法：

```Java
super.onStart(intent);
super.setUIContent(ResourceTable.Layout_ability_main);

String token_s =  DataBaseUtil.getToken(this);
//如果本地数据库没有token，说明用户还没有登陆
if (token_s == null) {
    Button btn_login = findComponentById(ResourceTable.Id_login_btn);
    TextField tf_userid = findComponentById(ResourceTable.Id_login_id_textfield);
    TextField tf_userPwd = findComponentById(ResourceTable.Id_login_pwd_textfield);

    String url = "http://8.136.83.196:8899/users/login";
    btn_login.setClickedListener(component -> {
        //开新线程
        TaskDispatcher globalTaskDispatcher = this.getGlobalTaskDispatcher(TaskPriority.DEFAULT);
        //异步
        globalTaskDispatcher.asyncDispatch(() -> {
            String user_id = tf_userid.getText();
            String user_pwd = tf_userPwd.getText();
            //发送http请求，并获得数据
            Account account = new Account(user_id, user_pwd);
            String account_json = JSON.toJSONString(account);
            String login_msg = HttpRequestUtil.sendPostRequest(this, url, account_json);
            LoginMsg login_msg_obj = JSON.parseObject(login_msg, LoginMsg.class);
            String token = login_msg_obj.getToken();
            String msg = login_msg_obj.getMsg();
            if (token != null) {
                //将token存入本地数据库，并跳到个人页
                DataBaseUtil.setToken(token, this);
                present(new UserInfoAbilitySlice(), new Intent());
            } else {
                //返回主线程进行UI重绘，原因是show方法不能在子线程中运行
                getUITaskDispatcher().asyncDispatch(new Runnable() {
                    @Override
                    public void run() {
                        new ToastDialog(getContext()).setText(msg).show();
                    }
                });
            }
        });
    });
}
//如果本地数据库有token，说明已经登陆，就跳到个人页
else{
    present(new UserInfoAbilitySlice(), new Intent());
}
```

- 进来先进行一个判断，若本地数据库里有token，则直接跳转到个人页，为了应对APP被杀死后重启的情况。
- 按钮监听，获取输入框的信息，向后端发送请求，获得token和msg。
- 若token不为空，则存到本地数据库，并跳转到个人页。
- 若token为空，则建立一个ToastDialog组件，显示msg。ToastDialog组件专门用于显示提示信息，它存在几秒后自动消失。

{{< admonition type=info title="注意" open=true >}}
建立Toast Dialog组件这里，重绘UI的操作只能在主线程里运行，在这里指show方法，如果直接写`new ToastDialog(getContext()).setText(msg).show();`是不行的，因为此时我们正在新建的线程里，这个任务要扔回到主线程，所以才有了以下的代码块。
{{< /admonition >}}

```java
getUITaskDispatcher().asyncDispatch(new Runnable() {
    @Override
    public void run() {
        new ToastDialog(getContext()).setText(msg).show();
    }
});
```

onActive方法：

```java
super.onActive();
//程序重新返回前台调用
//若已经登陆，则导航到个人页
String token_s =  DataBaseUtil.getToken(this);
if (token_s != null){
    present(new UserInfoAbilitySlice(), new Intent());
}
```

当页面从后台返回前台时，调用的是onActive方法，比如用户在导航到个人页之后，点了一下退出键，就会返回默认页，我们不想让他再发一次http请求，检查token，若存在，直接将其导航回个人页。

## UserInfoAbilitySlice

onStart方法：

```Java
super.onStart(intent);
super.setUIContent(ResourceTable.Layout_ability_userinfo);
//能导航到此页说明用户已经登陆，向服务器请求用户的头像
String token = DataBaseUtil.getToken(this);
Image image = findComponentById(ResourceTable.Id_image);
if (token != null){
    //建新线程
    TaskDispatcher globalTaskDispatcher = this.getGlobalTaskDispatcher(TaskPriority.DEFAULT);
    //异步
    globalTaskDispatcher.asyncDispatch(()->{
        //发送请求，更新image组件
        String url = "http://8.136.83.196:8899/users/info";
        String img_url = HttpRequestUtil.sendPostRequestWithToken(this,url,token);
        LoadImageUtil.loadImg(this,img_url,image);
    });
}
```

- 导航到此页时，从本地数据库中取出token，向后端发送。
- 后端返回头像的url。
- 将url显示到image组件。

## 一些瑕疵

1. 没有退出登陆键，这个很容易实现，点击按钮，把token删掉即可，我懒得写了。
2. 个人页太简单，懒就一个字。

## 结束语

### 源码

[https://github.com/leeshy-tech/HarmonyOS_example/tree/main/Users](https://github.com/leeshy-tech/HarmonyOS_example/tree/main/Users)

### 参考文献

[HarmonyOS 2.0应用开发实战教程丨锋迷商城项目](https://www.bilibili.com/video/BV1DM4y1G7MN)

[HarmonyOS文档](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/document-outline-0000001064589184)	

