# 鸿蒙开发实践——调用API


## 项目简介

### 项目结构

```
└─ entry
	├─src.main
        ├─ java.com.example.api
            ├─beans
                ├─ requestBody				API请求体
                └─ returnBody				API返回体
            ├─slice
                └─ MainAbilitySlice			主页面Slice
            ├─utils
                ├─ baiduApi					百度API调用
                ├─ HttpRequestUtil			一般API调用，本项目没用到，但是很重要
                └─ LoadImageUtil			网络图片加载到image组件
            ├─ MainAbility
            └─ MyApplication
        └─ resources.base.layout
            └─ ability_main.xml				主页面布局文件
 	└─ build.gradle							添加外部依赖
```

### 效果

单击按钮，将输入框内的文字转换为二维码显示在下方。

用手机扫码，解码的信息与文字相同。

![20220212_233555](/image/HarmonyOS/20220212_233555.gif)

## 铺垫

### HTTP

{{< admonition type=info title="注意" open=true >}}
本项目中调用API使用的是https方式，这部分仅作笔记。
{{< /admonition >}}

在鸿蒙应⽤中默认只允许访问https接⼝，如果要访问http接⼝，需要在config.json的deviceConfig项中添加如下配置：

```json
"deviceConfig": {
  "default": {
    "network": {
      "cleartextTraffic": true
    }
  }
}
```

### JSON

API请求体的传入和响应体的解析需要JSON格式的字符串、把JSON字符串转换为对象，虽然也可以用转义符等方式实现，但还是用外部依赖比较方便，常用的依赖有Gson、FastJson、Jackson，本项目使用的是FastJson。

借助转义符传递JSON be like：

```Java
String jsonBody = "{\n  \"data\" : \"https://apis.baidu.com/\",\n" +
        "  \"size\" : 20,\n" +
        "  \"level\" : \"L\",\n" +
        "  \"format\" : \"jpg\",\n" +
        "  \"logo\" : \"https://apisown-test.bj.bcebos.com/qr-code-api-store.png\"\n}";
```

我的老天，它实在是太丑陋了。

### 引入外部依赖

添加依赖库fastjson，以及百度的依赖库api-explorer-sdk。

在entry > build.gradle中添加：`implementation ('com.baidubce:api-explorer-sdk:1.0.3.1','com.alibaba:fastjson:1.2.47')`

{{< admonition type=tip title="技巧" open=true >}}
添加外部依赖的方法，详见：[鸿蒙开发笔记——引入外部依赖](https://leeshy-tech.github.io/harmonyos_outerdependency/)
{{< /admonition >}}

## API调用

API的功能是将字符串转化为二维码图片，这里选用的是[百度的API](https://apis.baidu.com/store/detail/581576df-bc52-4e4a-8a3a-2abd6035e7ae)，原因是：有例程。对于我这种啥都没学扎实的人来说，能降低点难度最好。

### API需要注意的点

1. 调用方式：HTTP or HTTPS? POST？GET？DELETE？
2. 请求体参数。
3. 响应体参数：取决于你如何取到你想要的返回值。
4. 认证密钥。

从该API的介绍中我们看到：http和https均可，POST方式，有认证密钥，请求体和响应体参数示例如下：

![image-20220212215401143](/image/HarmonyOS/image-20220212215401143.png)![image-20220212215413369](/image/HarmonyOS/image-20220212215413369.png)

### 核心代码

1. 请求体和响应体各写成一个类：requestBody 、returnBody

   ```Java
   //API的请求体，共五个参数
   public class requestBody {
       private String data;
       private int size;
       private String level;
       private String format;
       private String logo;
   
      ...
   }
   ```
   
   ```Java
    //API的响应体类，只有一个参数
    public class returnBody {
        private String imageUrl;
    
        ...
    }
   ```


2. 调用API的过程封装成类的函数：baiduApi.sendRequest(requestBody request_body)

   ```Java
   //百度API的请求代码，由示例代码更改而来：https://apis.baidu.com/store/detail/581576df-bc52-4e4a-8a3a-2abd6035e7ae
   public static String sendRequest(requestBody request_body) {
       //填入自己的accessKey，secretKey，否则项目无法正常运行。
       String accessKey = "accessKey";
       String secretKey = "secretKey";
   
       String result = null;
       String path = "http://qrcode.api.bdymkt.com/qrcode/generate";
       ApiExplorerRequest request = new ApiExplorerRequest(HttpMethodName.POST, path);
       request.setCredentials(accessKey, secretKey);
   
       // 设置header参数
       request.addHeaderParameter("Content-Type", "application/json;charset=UTF-8");
   
       // 设置jsonBody参数
       String objStr = JSON.toJSONString(request_body);
       request.setJsonBody(objStr);
   
       ApiExplorerClient client = new ApiExplorerClient(new AppSigner());
   
       try {
           ApiExplorerResponse response = client.sendRequest(request);
           // 返回结果格式为Json字符串
           result = response.getResult();
       } catch (Exception e) {
           e.printStackTrace();
       }
       return result;
   }
   ```

## 按键监听

{{< admonition type=info title="注意" open=true >}}
本部分参考官方文档 > Ability框架 > 线程管理
{{< /admonition >}}

这里的按键监听与之前稍有不同，原因是调用API是一个耗时的工作，它不能在主线程中运行，需要在按键监听器中使用新的线程。

### 核心代码

MainAbilitySlice的onStart方法：

```Java
//设置按键监听
btn1.setClickedListener(component -> {
    //开一个新线程
    TaskDispatcher globalTaskDispatcher = this.getGlobalTaskDispatcher(TaskPriority.DEFAULT);
    //异步
    globalTaskDispatcher.asyncDispatch(()->{
        //调用API生成二维码图片（网络地址）
        //返回字符串格式：{"imageUrl":"https://bj.bcebos.com/qr-code/22021215e07535dcaa53.jpg"}
        String string = tf1.getText();
        requestBody request_body = new requestBody(string,20,"L","jpg",
                "https://apisown-test.bj.bcebos.com/qr-code-api-store.png");
        String request_result = baiduApi.sendRequest(request_body);
        //将JSON字符串转换为类，取出imageUrl
        returnBody returndata = JSON.parseObject(request_result, returnBody.class);
        String image_url = returndata.getImageUrl();
        //将网络图片显示到image组件
        LoadImageUtil.loadImg(this,image_url,image1);
    });
});
```

在新线程的异步方法里写入监听逻辑：调用API并将返回的网络图片url显示到image组件。

## 网络图片显示

{{< admonition type=info title="注意" open=true >}}
本部分参考官方文档 > 媒体 > 图像 > 位图操作开发指导 & 图像解码开发指导。
{{< /admonition >}}

当我们API调用成功之后，我们就需要显示这个图片，但是image组件的setImageElement()方法的输入参数类型只能是Element，而Element类型是鸿蒙的本地数据文件管理类型，也就是说通过这个方法只能让image组件显示本地的图片，我们获得的网络图片地址在这里是不能用的。

![image-20220212224107044](/image/HarmonyOS/image-20220212224107044.png)

我们需要自己写接口来实现网络图片的显示：LoadImageUtil.loadImg(Context context, String netImgUrl, Image image)

```java
//将网络图片加载到context的image组件里
public static void loadImg(Context context, String netImgUrl, Image image){
    //创建一个新线程
    TaskDispatcher globalTaskDispatcher = context.getGlobalTaskDispatcher(TaskPriority.DEFAULT);
    globalTaskDispatcher.asyncDispatch(()->{
        HttpURLConnection connection = null;
        try{
            //建立与网络图片之间的http连接
            URL url = new URL(netImgUrl);
            connection = (HttpURLConnection) url.openConnection();
            connection.connect();
            //从连接中获取输入流
            InputStream inputStream = connection.getInputStream();
            //根据数据流将图片数据缓存到ImageSouce对象，创建图片对象
            ImageSource imageSource = ImageSource.create(inputStream,new ImageSource.SourceOptions());
            //图片数据解码的参数
            ImageSource.DecodingOptions decodingOptions = new ImageSource.DecodingOptions();
            decodingOptions.desiredPixelFormat = PixelFormat.ARGB_8888;
            //PixelMap对象就表示一个图片
            PixelMap pixelmap = imageSource.createPixelmap(decodingOptions);
            //将图片载入到组件中：在鸿蒙应用中将图片载入到组件，推荐在一个独立的UI线程中完成
            context.getUITaskDispatcher().asyncDispatch(()->{
                image.setPixelMap(pixelmap);
                pixelmap.release();//释放图片
            });
        }catch (IOException e){
            e.printStackTrace();
        }
    });
}
```

主要思路：

- 建立网络连接获得图片数据
- 图片数据解码
- 转换成位图对象pixelmap
- 通过image.setPixelMap(pixelmap)方法载入图片

## 结束语

### 项目地址

[https://github.com/leeshy-tech/HarmonyOS_example/tree/main/Api](https://github.com/leeshy-tech/HarmonyOS_example/tree/main/Api)

### 参考文献

[Fastjson 使用实例](https://www.w3cschool.cn/fastjson/fastjson-demo1.html)

[Java 中 JSON 的使用](https://www.runoob.com/w3cnote/java-json-instro.html)

[百度智能云——二维码生成识别](https://apis.baidu.com/store/detail/581576df-bc52-4e4a-8a3a-2abd6035e7ae)

[HarmonyOS文档](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/document-outline-0000001064589184)	

[HarmonyOS 2.0应用开发实战教程丨锋迷商城项目](https://www.bilibili.com/video/BV1DM4y1G7MN)


