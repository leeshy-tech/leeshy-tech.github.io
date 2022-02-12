# 鸿蒙开发笔记——引入外部依赖


## Gradle

简单的说，Gradle是一个构建工具，它是用来帮助我们构建app的，构建包括编译、打包等过程。我们可以为Gradle指定构建规则，然后它就会根据我们的“命令”自动为我们构建app。Studio中默认就使用Gradle来完成应用的构建。在创建工程时，Studio自动帮我们生成了一些通用构建规则，很多时候我们甚至完全不用修改这些规则就能完成我们app的构建。

有些时候，我们会有一些个性化的构建需求，比如我们引入了第三方库，或者我们想要在通用构建过程中做一些其他的事情，这时我们就要自己在系统默认构建规则上做一些修改。这时候我们就要自己向Gradle”下命令“了，这时候我们就需要用Gradle能听懂的话了，也就是Groovy。Groovy是一种基于JVM的动态语言，关于它的具体介绍，感兴趣的同学可以文末参考”延伸阅读“部分给出的链接。

## 依赖坐标

### 什么是依赖坐标

依赖坐标的概念来源于Maven，俗称 gav：指的是使用下面三个向量子仓库中唯一定位一个 Maven 工程。

1. groupid:公司或组织域名倒序 

   <groupid>com.ys.maven</groupid>

2. artifactid:模块名，也是实际项目的名称

   <artifactid>Maven_05</artifactid>

3. version:当前项目的版本

   <version>0.0.1-SNAPSHOT</version>

### 如何获取依赖坐标

1. [https://mvnrepository.com/](https://mvnrepository.com/) 在mvnrepository官网查询

   ![image-20220212144009395](/image/HarmonyOS/image-20220212144009395.png)

   单击版本号，可以看到它的坐标信息：

   ![image-20220212144058814](/image/HarmonyOS/image-20220212144058814.png)

2. 各种提示：

   比如我现在要调用百度公司的API，示例代码提示我要引用百度的依赖：

   ![image-20220212144250492](/image/HarmonyOS/image-20220212144250492.png)

   那么就获取了坐标信息。

## 在鸿蒙应用中引用依赖

{{< admonition type=note title="注意" open=true >}}
虽然我们在坐标依赖部分一直在说Maven，但是在DevEco Studio中使用的还是gradle，这点要分清楚。
{{< /admonition >}}

1. 打开entry > build.gradle：

	![image-20220212144543246](/image/HarmonyOS/image-20220212144543246.png)

2. 在dependencies一栏添加如下语句：`implementation ('依赖坐标1','依赖坐标2','依赖坐标3')`

   依赖坐标 = `groupid:artifactid:version`

   例如：
   ```groovy
   dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar', '*.har'])
    testImplementation 'junit:junit:4.13.1'
    ohosTestImplementation 'com.huawei.ohos.testkit:runner:2.0.0.200'
    implementation ('com.google.code.gson:gson:2.8.8','com.baidubce:api-explorer-sdk:1.0.3.1')
   }
   ```

3. 打开gradle页面，刷新。

   ![image-20220212145129593](/image/HarmonyOS/image-20220212145129593.png)

   ![image-20220212145154030](/image/HarmonyOS/image-20220212145154030.png)

4. 左侧工程目录能看到加入的依赖

   ![image-20220212145345190](/image/HarmonyOS/image-20220212145345190.png)
   
5. 由此，在工程中能够正常使用外部依赖。

   ![image-20220212145858412](/image/HarmonyOS/image-20220212145858412.png)

## 结束语

### 参考文献

[十分钟理解Gradle](https://www.cnblogs.com/Bonker/p/5619458.html)

[https://mvnrepository.com/](https://mvnrepository.com/)

[Maven详解（五）------ 坐标的概念以及依赖管理](https://www.cnblogs.com/ysocean/p/7451054.html)

[HarmonyOS 2.0应用开发实战教程丨锋迷商城项目](https://www.bilibili.com/video/BV1DM4y1G7MN)

