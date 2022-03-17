# Linux——图片url生成


# 图片url生成

## 准备

### 安装JDK，JRE

- 下载linux的jdk，jre压缩包：

  - [https://www.oracle.com/java/technologies/downloads/#java8](https://www.oracle.com/java/technologies/downloads/#java8)
  - 用xftp传到服务器上。
  - 解压`tar -zxvf jdk-17_linux-x64_bin.tar.gz jre-8u321-linux-x64.tar.gz`，记住此时所在的路径（pwd命令可查）。

- 配置环境变量：

  - `cd /etc`

  - `vim profile`

  - i进入编辑模式

  - 文件末尾加上如下代码，JAVA_HOME,JRE_HOME分别为解压的文件夹的路径：

    ```shell
    export JAVA_HOME=/home/ubuntu/jdk-17.0.2
    export JRE_HOME=/home/ubuntu/jre1.8.0_321
    export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib
    export PATH=$JAVA_HOME/bin:$PATH
    ```

  - `Esc`退出，输入`:wq`保存退出。

- 使能配置：`source /etc/profile`

- 检验：`java -version`，输出版本信息，安装成功。

### 安装tomcat

- 下载安装包：

  - [https://tomcat.apache.org/download-10.cgi](https://tomcat.apache.org/download-10.cgi)

    ![image-20220317165458557](/image/Linux/image-20220317165458557.png)

  - 用xftp传到服务器上，解压`tar -zxvf apache-tomcat-10.0.18.tar.gz`

- 进入bin目录，启动tomcat

  - `cd apache-tomcat-10.0.18/bin`

  - `./startup.sh`

    ![image-20220317170011765](/image/Linux/image-20220317170011765.png)

### 防火墙设置

到服务器控制台，开启8080端口：

![image-20220317170703321](/image/Linux/image-20220317170703321.png)

浏览器访问`服务器IP:8080`地址，看到tomcat默认页说明tomcat安装运行成功。

![image-20220317200700408](/image/Linux/image-20220317200700408.png)

## 生成图片url

- 假设图片存放在服务器路径：`/home/ubuntu/image`

- 进入tomcat文件夹的conf路径，编辑server.xml：

  - `cd conf`

  - `vim server.xml`

  - 结尾处插入此代码`<Context docBase="/home/ubuntu/image" path="/pictures" debug="0" reloadable="true"/>`

    ![image-20220317201524598](/image/Linux/image-20220317201524598.png)

则可以通过`url=http://ip:8080/path/图片名+后缀`访问该图片。

![image-20220317204437349](/image/Linux/image-20220317204437349.png)

## 结束语

### 参考文献

[https://www.bilibili.com/video/BV1uh411a7Jg?p=268](https://www.bilibili.com/video/BV1uh411a7Jg?p=268)

[tomcat启动“成功”，但是浏览器无法访问](https://blog.csdn.net/dyt443733328/article/details/102587168)

[访问 Linux 服务器上的文件（以图片为例](https://blog.csdn.net/Wyx_wx/article/details/89117746)

