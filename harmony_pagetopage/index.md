# 鸿蒙开发实践——页面导航

# 页面导航
## 项目简介

### 项目结构

```
├─ entry.src.main
	├─ com.example.page_to_page
        ├─slice
            ├─ AnotherAbilitySlice
            ├─ MainAbilitySlice
            └─ SecondAbilitySlice
        ├─ AnotherAbility	
        ├─ MainAbility
        └─ MyApplication
	└─ resources.base.layout
        ├─ ability_another.xml
        ├─ ability_main.xml
        └─ ability_second.xml
```

### 效果

点击按钮一，从MainAbilitySlice跳转到SecondAbilitySlice。

点击按钮二，从MainAbilitySlice跳转到SecondAbilitySlice，并传递参数字符串。

点击按钮三，从MainAbility的MainAbilitySlice跳转到AnotherAbility的AnotherAbilitySlice。

![20220128_225821](/image/PageToPage/20220128_225821.gif)


## intent

> [HarmonyOS文档——intent](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/ability-intent-0000000000038799)

intent是对象之间传递信息的载体，Slice之间的导航、传参以及Ability之间的导航都是基于intent。Intent的构成元素包括Operation与Parameters。Operation是执行的操作，Parameters则是携带的参数。

## Slice间导航

核心代码：

MainAbilitySlice的onStart方法：

```Java
Button btn1 = (Button) findComponentById(ResourceTable.Id_btn1);
btn1.setClickedListener(listener->present(new SecondAbilitySlice(),new Intent()));
```

- 通过id获取按钮对象。
- 给按钮绑定事件监听器，执行present方法。
- 这里只是导航，没有其他操作，所以传递一个默认intent即可。

## Slice间传参

核心代码：

```Java
Button btn2 = (Button) findComponentById(ResourceTable.Id_btn2);
btn2.setClickedListener(listener -> {
    Intent intent1 = new Intent();
    intent1.setParam("my_string","从MainAbilitySlice传参");
    this.present(new SecondAbilitySlice(),intent1);
});
```

跟导航部分思路相同。

传参的关键是构造intent对象的Parameters属性，使用setParam方法存储键值对。setParam方法有很多重载，包括int，string等等，但是没有对象类型，也就是传参不能传对象。

## PageAbility间导航

核心代码：

MainAbilitySlice类的onStart方法：

```Java
Button btn3 = (Button) findComponentById(ResourceTable.Id_btn3);
btn3.setClickedListener(listener -> navigateToAnotherPage(listener));
```

设置监听器的逻辑相同，不过这次我们让监听器执行我们的自定义函数navigateToAnotherPage。

MainAbilitySlice类新增:

```java
private void navigateToAnotherPage(Component component){
        Intent intent = new Intent();
        Operation operation = new Intent.OperationBuilder()
                .withDeviceId("")						//空字符串为本机
                .withBundleName("com.example.page_to_page")//本应用的标识
                .withAbilityName("com.example.page_to_page.AnotherAbility")//想启动的Ability
                .build();

        intent.setOperation(operation);
        this.startAbility(intent);
    }
```

- 使用OperationBuilder构建一个Operation，设置给intent。
- 将intent传给监听器

页面跳转的核心是intent对象的Operation属性，这里构建Operation有三个参数DeviceId、BundleName、AbilityName，因为鸿蒙可以启动任意设备的任意应用的任意Ability，可能这就是万物互联吧。

## 结束语

### 项目源码

[https://github.com/leeshy-tech/HarmonyOS_example/tree/main/page_to_page](https://github.com/leeshy-tech/HarmonyOS_example/tree/main/page_to_page)

## 参考文献

[HarmonyOS 2.0应用开发实战教程丨锋迷商城项目](https://www.bilibili.com/video/BV1DM4y1G7MN)

[HarmonyOS文档——intent](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/ability-intent-0000000000038799)

