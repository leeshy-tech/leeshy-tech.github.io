# 鸿蒙开发实践——ServiceAbility

# ServiceAbility实践
## 项目简介

### 项目结构

```
├─ entry.src.main
	├─ com.example.serviceability
        ├─slice
            └─ MainAbilitySlice
        ├─ MainAbility	
        ├─ MyService
        └─ MyApplication
	└─ resources.base.layout
        └─ ability_main.xml
```

### 效果
按钮1开启MyService

按钮2连接到MyService

按钮3断开与MyService的连接

按钮4关闭MyService

不同的点击顺序，命令行输出的提示信息不同。	

## 铺垫

在MyService里的每个生命周期函数里都加一句sout来显示各个方法执行的顺序。

```java
public class MyService extends Ability {
    private static final HiLogLabel LABEL_LOG = new HiLogLabel(3, 0xD001100, "Demo");

    //在每个生命周期函数内加一句sout调试
    @Override
    public void onStart(Intent intent) {
        super.onStart(intent);
        HiLog.error(LABEL_LOG, "MyService::onStart");
        System.out.println("--------------------onStart");
    }

    @Override
    public void onBackground() {
        super.onBackground();
        HiLog.info(LABEL_LOG, "MyService::onBackground");
        System.out.println("--------------------onBackground");
    }

    @Override
    public void onStop() {
        super.onStop();
        HiLog.info(LABEL_LOG, "MyService::onStop");
        System.out.println("--------------------onStop");
    }

    @Override
    public void onCommand(Intent intent, boolean restart, int startId) {
        System.out.println("--------------------onCommand");
    }

    @Override
    public IRemoteObject onConnect(Intent intent) {
        System.out.println("--------------------onConnect");
        return new LocalRemoteObject() {};
    }

    @Override
    public void onDisconnect(Intent intent) {
        System.out.println("--------------------onDisconnect");
    }
}
```

## 开启和关闭ServiceAbility

> 此部分和”页面导航“里开启Ability的操作如出一辙，也就是说，我们只是在开启和关闭Ability，至于它是什么类型，无所谓。

核心代码：

MainAbilitySlice的onStart方法：

```java
//按钮1开启MyService
Button btn1 = (Button) findComponentById(ResourceTable.Id_btn1);
btn1.setClickedListener(component -> {
    Intent intent1 = new Intent();
    Operation operation = new Intent.OperationBuilder()
            .withDeviceId("")
            .withBundleName("com.example.serviceability")
            .withAbilityName("com.example.serviceability.MyService")
            .build();
    intent1.setOperation(operation);
    this.startAbility(intent1);
});
```

```java
//按钮4关闭MyService
Button btn4 = (Button) findComponentById(ResourceTable.Id_btn4);
btn4.setClickedListener(component -> {
    Intent intent3 = new Intent();
    Operation operation = new Intent.OperationBuilder()
            .withDeviceId("")
            .withBundleName("com.example.serviceability")
            .withAbilityName("com.example.serviceability.MyService")
            .build();
    intent3.setOperation(operation);
    this.stopAbility(intent3);
});
```

- 通过id获取Button对象，设置事件监听器。
- 调用**startAbility**和**stopAbility**方法，在intent对象的Operation属性里指定开启哪台**设备**的哪个**应用**的哪个**Ability**。

## 建立连接

核心代码：

MainAbilitySlice的onStart方法：

```java
//按钮2连接到MyService
Button btn2 = (Button) findComponentById(ResourceTable.Id_btn2);

IAbilityConnection connection = new IAbilityConnection() {
    @Override
    public void onAbilityConnectDone(ElementName elementName, IRemoteObject iRemoteObject, int i) {
        System.out.println("----------------------连接MyService成功");
    }

    @Override
    public void onAbilityDisconnectDone(ElementName elementName, int i) {
        System.out.println("----------------------连接MyService失败");
    }
};

btn2.setClickedListener(component -> {
    Intent intent2 = new Intent();
    Operation operation = new Intent.OperationBuilder()
            .withDeviceId("")
            .withBundleName("com.example.serviceability")
            .withAbilityName("com.example.serviceability.MyService")
            .build();
    intent2.setOperation(operation);
    this.connectAbility(intent2,connection);
});
```

- 通过id获取Button对象，设置事件监听器。
- 新建连接对象，重写onAbilityConnectDone和onAbilityDisconnectDone方法，每个方法里都写一句sout用于调试。
  - onAbilityConnectDone：连接成功建立后执行。
  - onAbilityDisconnectDone：连接建立失败后执行。

- 调用connectAbility方法，传递intent和connect对象。

MyService：

```java
@Override
public IRemoteObject onConnect(Intent intent) {
    System.out.println("--------------------onConnect");
    return new LocalRemoteObject() {};
}
```

- 注意返回语句，返回一个LocalRemoteObject对象。

试图与Service建立连接时，触发onConnect方法，它返回一个LocalRemoteObject对象，在这个实例中它返回的是MyService这个Ability，触发回调函数onAbilityConnectDone或者onAbilityDisconnectDone。

## 关闭连接

核心代码：

MainAbilitySlice的onStart方法：

```java
//按钮3断开与MyService的连接
Button btn3 = (Button) findComponentById(ResourceTable.Id_btn3);
btn3.setClickedListener(component -> {
    if(connection != null){
        this.disconnectAbility(connection);
    }
});
```

- 若connection对象存在，就调用**disconnectAbility**方法即可。

## 生命周期分析

整个执行过程如图所示：

![image-20220129222251526](/image/ServiceAbility/image-20220129222251526.png)

- 启动：
  - 若MyService已建立，执行onCommand。
  
    ![image-20220129225314667](/image/ServiceAbility/image-20220129225314667.png)
  
  - 若未建立，执行onStart和onCommand。
  
    ![image-20220129222803130](/image/ServiceAbility/image-20220129222803130.png)
  
- 连接：
  - 若MyService已建立，则执行onConnect。
  
    ![image-20220129222853220](/image/ServiceAbility/image-20220129222853220.png)
  
  - 若未建立，则执行onStart和onConnect（红字忽略）。
  
    ![image-20220129223038075](/image/ServiceAbility/image-20220129223038075.png)
  
- 断开连接：当连接存在，且
  - MyService是手动创建的，不是由连接唤起的，只执行onDisconnect
  
    ![image-20220129223338507](/image/ServiceAbility/image-20220129223338507.png)
  
  - MyService是由该连接唤起的，执行onDisconnect、onBackground、onStop。
  
    ![image-20220129223108832](/image/ServiceAbility/image-20220129223108832.png)
  
- 关闭：当MyService没有被连接时，才能关闭，执行onBackground和onStop。

  ![image-20220129223151020](/image/ServiceAbility/image-20220129223151020.png)

## 结束语

### 项目源码

[https://github.com/leeshy-tech/HarmonyOS_example/tree/main/ServiceAbility](https://github.com/leeshy-tech/HarmonyOS_example/tree/main/ServiceAbility)

### 参考文献

[HarmonyOS 2.0应用开发实战教程丨锋迷商城项目](https://www.bilibili.com/video/BV1DM4y1G7MN)

[HarmonyOS文档——ServiceAbility](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/ability-service-concept-0000000000044457)


