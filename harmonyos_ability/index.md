# 鸿蒙开发笔记——Ability


# Ability

{{< admonition type=quote title="HarmonyOS文档" open=true >}}

Ability是应用所具备能力的抽象，也是应用程序的重要组成部分。一个应用可以具备多种能力，HarmonyOS支持应用以Ability为单位进行部署。Ability可以分为FA（Feature Ability）和PA（Particle Ability）两种类型，每种类型为开发者提供了不同的模板，以便实现不同的业务功能。

 {{< /admonition >}}

总的来说，Ability是功能相似的内容的集合，这种划分便于应用的编程组织，FA提供与用户交互的能力，只包含PageAbility，PA没有可视化界面，包括ServiceAbility和DataAbility。

## 配置

一个应用中所有的Ability必须在config.json中注册。

如果通过新建Ability来创建Ability，Studio会自动注册，不需要手动更改config.json。

但如果通过新建Java类来创建，则需要在config.json添加相关的配置信息。

![image-20220208214059300](/image/Ability/image-20220208214059300.png)

同样，当删除Ability时，需要删除config.json中对应的配置信息，否则会出现一些问题。

## 生命周期

PageAbility和ServiceAbility具有复杂的生命周期，创建、隐藏到后台、从后台呼出等等都会执行相关的生命周期函数，其中onStart方法最常用，当Ability创建时调用onStart方法，所以UI的绑定、按钮事件响应器等等都需要在onStart方法中实现。

![PageAbility生存周期](/image/Ability/生存周期.png)

## Intent

{{< admonition type=quote title="HarmonyOS文档" open=true >}}

Intent是对象之间传递信息的载体。例如，当一个Ability需要启动另一个Ability时，或者一个AbilitySlice需要导航到另一个AbilitySlice时，可以通过Intent指定启动的目标同时携带相关数据。

 {{< /admonition >}}

Intent是页面跳转及传参的关键，他包括Operation与Parameters两个属性，不传参时只需要构造默认intent，例如下：

```java
btn1.setClickedListener(component -> {
 	Intent intent = new Intent();
 	this.present(new SecondAbilitySlice(),intent);
 });
```

参数通过Parameters来传递：

```java
btn1.setClickedListener(component -> {
     Intent intent1 = new Intent();
     intent1.setParam("productId","101");
     this.present(new SecondAbilitySlice(),intent1);
 });
```

通过设置Operation可以启动任意**设备**的任意**应用**的任意**Ability**：

```java
Intent intent = new Intent();
// 通过Intent中的OperationBuilder类构造operation对象，指定设备标识（空串表示当前设备）、应用包名、Ability名称
Operation operation = new Intent.OperationBuilder()
        .withDeviceId("")
        .withBundleName("com.demoapp")
        .withAbilityName("com.demoapp.FooAbility")
        .build();
// 把operation设置到intent中
intent.setOperation(operation);
startAbility(intent);
```

## PageAbility

一个PageAbility可以包含任意个AbilitySlice，默认展示的AbilitySlice是通过**setMainRoute()**方法来指定的。	

### AbilitySlice

AbilitySlice相当于一个页面，其显示的内容是通过组件来声明的，其组件加载⽀持两种⽅式：

- Java代码
- xml布局文件

在其onStart方法中通过 setUIContext 来加载视图组件，它有两个重载：

- setUIContext(int) : 通过布局⽂件的ID，加载resources/base/layout⽬录下的布局⽂件完成⻚⾯的渲染。
- setUIContext(ComponentContainer) :通过加载⼀个使⽤Java代码创建的组件完成⻚⾯的渲染 。

## ServiceAbility

{{< admonition type=quote title="HarmonyOS文档" open=true >}}

基于Service模板的Ability（以下简称“Service”）主要用于后台运行任务（如执行音乐播放、文件下载等），但不提供用户交互界面。Service可由其他应用或Ability启动，即使用户切换到其他应用，Service仍将在后台继续运行。

 {{< /admonition >}}

### 前台Service

一般情况下，Service都是在后台运行的，后台Service的优先级都是比较低的，当资源不足时，系统有可能回收正在运行的后台Service。

在一些场景下（如播放音乐），用户希望应用能够一直保持运行，此时就需要使用前台Service。前台Service会始终保持正在运行的图标在系统状态栏显示。

## DataAbility

{{< admonition type=quote title="HarmonyOS文档" open=true >}}

使用Data模板的Ability（以下简称“Data”）有助于应用管理其自身和其他应用存储数据的访问，并提供与其他应用共享数据的方法。Data既可用于同设备不同应用的数据共享，也支持跨设备不同应用的数据共享。

数据的存放形式多样，可以是数据库，也可以是磁盘上的文件。Data对外提供对数据的增、删、改、查，以及打开文件等接口，这些接口的具体实现由开发者提供

 {{< /admonition >}}

### URI

URI用来标识一个具体的数据，例如数据库中的某个表或磁盘上的某个文件。格式如下：

![image-20220208221232345](/image/Ability/image-20220208221232345.png)

- scheme：协议方案名，固定为“dataability”，代表Data Ability所使用的协议类型。
- authority：设备ID。如果为跨设备场景，则为目标设备的ID；如果为本地设备场景，则不需要填写。
- path：资源的路径信息，代表特定资源的位置信息。
- query：查询参数。
- fragment：可以用于指示要访问的子资源。

URI示例：

- 跨设备场景：dataability://*device_id*/*com.domainname.dataability.persondata*/*person*/*10*
- 本地设备：dataability:///*com.domainname.dataability.persondata*/*person*/*10*

### 数据操作

DataAbility可以对文件或数据库进行数据操纵，不同类型数据管理方式写法都不一样，详见文档的**数据管理**一栏：

![image-20220208221629384](/image/Ability/image-20220208221629384.png)

## 实践工程

[页面导航](https://leeshy-tech.github.io/harmonyos_pagetopage/)

[ServiceAbility](https://leeshy-tech.github.io/harmonyos_serviceability/)

[电话簿](https://leeshy-tech.github.io/harmonyos_addressbook/)

## 结束语

本文仅是对官方文档做一个简单的总结+个人理解，详细的内容还需参考官方文档。

### 参考文献

[HarmonyOS——文档](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/document-outline-0000001064589184)

