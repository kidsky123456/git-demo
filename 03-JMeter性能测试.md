

# 三日速成：精通Jmeter接口测试

~~~properties
B站：https://www.bilibili.com/video/BV1pQyhYyEWf/?vd_source=f848549010351929c5d85f54b132fed2
---------------------
Python自动化测试视频：https://afglp.xet.tech/s/4vhNNM
笔记地址：https://pan.baidu.com/s/1nXEpYgxPSJlYUz3psRiDPg?pwd=w5i5 提取码：w5i5
---------------------
接口测试工具视频：https://afglp.xet.tech/s/2je8C4
笔记地址：https://pan.baidu.com/s/13l0maj9SVUFcGQDFUTUCqw?pwd=dknk 提取码：dknk
---------------------
jmeter安装包链接：https://pan.baidu.com/s/1aK5GtBpeVMaSVdZC1qUCFw?pwd=msxy 
提取码：msxy

jmeterplugins-standard链接：https://pan.baidu.com/s/1yw6V26c46jjxgi5wndY3Ig?pwd=msxy 
提取码：msxy 

jmeterplugins-extras链接：https://pan.baidu.com/s/1Mu6sPgedRQ67OUHUnVrJAw?pwd=msxy 
提取码：msxy
~~~

软件测试
横向：**软件测试**、银行测试、车载测试、大数据测试、嵌入式测试
纵向：功能测试、**接口测试**、自动化测试、性能测试、测试开发

~~~properties
软件测试和接口测试是目前主流的测试
~~~

## 1、接口和接口测试

【接口】：是【系统】内外交互的方式

- 接受来自用户的数据（内容），返回来自系统的响应
- 完成数据的校验、加工、存储、权限控制

接口例子：

- 电脑 有各种接口（USB、3.5mm音频）
- 电路板 有各种金属焊点（跳线）
- 12306 有各种订票渠道（代售点、电话、网上、窗口）

接口的需求：

- 正确性需求
- 稳定性需求
- 友好性需求

接口的测试：

- 为什么要进行接口测试

- 接口测试周期

- 接口测试的重点

  1.服从接口文档

​       2.意外情况

接口测试工具：

- **JMeter：**支持多种接口类型，还能性能测试，开源，可进行二次扩展
- Postman：简单、方便，局限性比较大，适合开发临时性调试
- APIfox等：新工具，市场占有小，适合小团队尝鲜

~~~
思维导图：https://www.processon.com/view/link/64e729486ece22263c3e11de
~~~

接口的类型：

- **Restful：HTTP传输JSON**
- WebService: HTTP 传输xml
- RPC：HTTP 传输二进制



## 2、JMeter环境搭建

<img src="F:\西安用友-汇杰大数据7班\typora-user-images\image-20241211165828667.png" alt="image-20241211165828667" style="zoom:80%;" />

安装步骤：

1、解压运行 JDK17（已安装，可以不用管）

2、其余的软件，解压，记录位置即可

~~~properties
备注：把记录的位置添加到环境变量的后面即可
~~~

![image-20241211170159321](F:\西安用友-汇杰大数据7班\typora-user-images\image-20241211170159321.png)

### (1)JDK的安装以及环境变量的配置：

```properties
安装JDK:注意包括了JDK和JRE两个部分
环境变量：
    我的电脑右键属性-->高级系统设置-->高级-->环境变量-->系统变量：
    1、新建：
    变量名：JAVA_HOME
    变量值：C:\Program Files\Java\jdk1.8.0_211   (jdk的安装路径)
2、新建：
变量名：CLASSPATH
变量值：.;%JAVA_HOME%\lib\tools.jar;%JAVA_HOME%\lib\dt.jar; (固定的)
 
3、编辑PATH，在最前面添加：
%JAVA_HOME%\bin;(固定的)

环境验证：
在Dos窗口输入：java -version和javac
```

### (2)Jmeter的安装以及环境变量的配置：

~~~properties
安装Jmeter:压缩包不需要安装，直接到官网下载即可。
 
变量：
新增
变量名：JMETER_HOME
变量值：D:\apache-jmeter-5.2.1
 
CLASSPATH最后面:
变量值：%JMETER_HOME%\lib\ext\ApacheJMeter_core.jar;%JMETER_HOME%\lib\jorphan.jar;
 
PATH最前面加入:
变量值：%JMETER_HOME%\bin;
 
环境验证：
进入D:\apache-jmeter-5.2.1\bin，双击jmeter.bat，或在dos窗口输入jmeter命令打开jmeter界面，安装成功。
~~~

[2.jmeter搭建环境_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1pQyhYyEWf?vd_source=f848549010351929c5d85f54b132fed2&p=2&spm_id_from=333.788.player.switch)



## 3、JMeter结构体系



## 4、JMeter接口实战

