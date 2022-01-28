# 鸿蒙开发笔记——开发工具DevEco Studio


# DevEco Studio

> HUAWEI DevEco Studio（获取工具请[点击链接下载](https://developer.harmonyos.com/cn/develop/deveco-studio)，以下简称DevEco Studio）是基于IntelliJ IDEA Community开源版本打造，面向华为终端全场景多设备的一站式集成开发环境（IDE），为开发者提供工程模板创建、开发、编译、调试、发布等E2E的HarmonyOS应用/服务开发

## 配置

DevEco是基于IDEA打造的，所以二者非常相似，有些配置不会的话可以直接google IDEA的相关配置。 	

### 界面风格

> 将界面风格改为暗黑模式

- 打开设置：file>settings

- Appearance & Behavior > Appearance

- Theme：改为Darcula，点击OK。

  ![image-20220128163010388](/image/image-20220128163010388.png)

### 字体

- 打开设置：file>settings

- Editor > Font

- Font：字体风格，我个人比较喜欢Inconsolata ; Size：字号 ; Line height：行距

![image-20220128163911273](/image/image-20220128163911273.png)

### 注释颜色

> 初始的注释颜色非常浅，还是灰色，所以想调成亮一点的绿色。

- 打开设置：file>settings

- Editor > Color Scheme > Language Default
- 点开Comments，其中Block comment是块注释，Line comment是行注释，尽量二者颜色统一吧，颜色栏是可以复制的。

![image-20220128164213389](/image/image-20220128164213389.png)

### 输入联想——大小写不敏感

> 初始的输入联想是大小写敏感的，Str才能补全成String，设置大小写不敏感，使str也能补全成String。

- 打开设置：file>settings
- Editor > General > Code Completion
- 去掉右侧Match case的勾

![image-20220128165309298](/image/image-20220128165309298.png)

效果如下：

![image-20220128165458440](/image/image-20220128165458440.png)

### 自动引包优化

- 打开设置：file>settings

- Editor > General > Auto import

- 在右侧勾上如图的两句话。

  ![image-20220128165943342](/image/image-20220128165943342.png)

### 汉化

> 编辑器设中文其实意义不大，汉化部分很少，都是一些基础的单词，但是我看到中文有一种莫名的安心感，所以还是汉化了。

- 打开设置：file>settings
- Plugins
- 搜索chinese，在installed里找到Chinese (Simplified) ，点击enable。
- 注意你会在Marketplace里找到Chinese (Simplified) Language Pack，它是IDEA的插件，适配效果没有内置的好。

![image-20220128171448210](/image/image-20220128171448210.png)

## 项目结构

## 调试方法

鸿蒙项目有两种调试方式：previewer和在设备运行

#### Previewer

previewer本质上是个页面预览器，预览的页面是工作区前台的Page。位置在：

![image-20220128202357675](/image/image-20220128202357675.png)

#### 在设备运行

设备有很多种，本地模拟器、远程模拟器、远程真机等等（[细节可以看官方文档](https://developer.harmonyos.com/cn/docs/documentation/doc-guides/run_simulator-0000001053303709)），都是将整个项目完整的打包成应用在设备上运行。推荐使用远程模拟器，因为本地模拟器会占用相当大一部分的本机资源。在右上角打开设备管理器即可进行管理。

![image-20220128202313625](/image/image-20220128202313625.png)

### 选择调试方式

设备：不管是远程还是本地，使用上都会有一定的卡顿，并且项目具有默认显示页面，调试非默认页面时，需要保证跳转逻辑等等无误。

预览器：工作区打开什么文件就默认预览什么文件，不需要考虑其他页面或者Ability的逻辑，但是，预览器是高度阉割的，某些逻辑不能很好的在预览器运行，比如我昨天遇到一个报错`[ClassCastException: java.lang.Object cannot be cast to java.lang.String，在预览器中运行就会报这个错误无法启动预览器，但是在设备中运行就正常。所以要灵活选择调试方式。当然，不提供页面服务的Ability的肯定要在设备上测试。

## 结束语

### 参考文献

[黑马程序员鸿蒙开发系统教程]: https://www.bilibili.com/video/BV1LK4y1u7jZ?from=search&amp;seid=6660495365339355856&amp;spm_id_from=333.337.0.0
[HarmonyOS文档——工具简介]: https://developer.harmonyos.com/cn/docs/documentation/doc-guides/tools_overview-0000001053582387


