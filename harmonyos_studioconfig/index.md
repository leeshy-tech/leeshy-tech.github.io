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

## 结束语

### 参考文献

[黑马程序员鸿蒙开发系统教程]: https://www.bilibili.com/video/BV1LK4y1u7jZ?from=search&amp;seid=6660495365339355856&amp;spm_id_from=333.337.0.0
[HarmonyOS文档——工具简介]: https://developer.harmonyos.com/cn/docs/documentation/doc-guides/tools_overview-0000001053582387


