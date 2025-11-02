 

**目录**

[1、安装并启动jemeter](#t0)

[2、重点组件](#t1)

[2.1、线程组：](#t2)

[2.2、HTTP取样器​编辑](#t3)

[2.3、查看结果树](#t4)

[2.4、HTTP请求默认值](#t5)

[2.5、HTTP信息头管理器](#t6)

[2.6、JSON提取器](#t7)

[2.7、JSON断言](#t8)

[2.8、同步定时器](#t9)

[2.9、CSV数据文件设置](#t10)

[2.10、HTTP Cookie管理器](#t11)

[3、测试报告](#t12)

[4、性能分析](#t13)

[通过三大指标来分析性能问题：4.1、响应时间](#t14)

[4.2、错误率（可靠率）](#t15)

[4.3、吞吐量](#t16)

* * *

1、安装并启动[jemeter](https://so.csdn.net/so/search?q=jemeter&spm=1001.2101.3001.7020)
---------------------------------------------------------------------------------

法一：

![](https://i-blog.csdnimg.cn/direct/0c2cf3a7538548b5a60fac1f99cad01c.png)

法二：将文件路径复制一下，配置环境变量，打开cmd，输入jmeter就可以打开

2、重点组件
------

### 2.1、线程组：

添加一个线程组去管理所有的线程。

开发者工具：network表示监视网络

![](https://i-blog.csdnimg.cn/direct/0af648d3425d464ebbff8cc2067ad265.png)

点击XHR筛选出一些后端的接口

![](https://i-blog.csdnimg.cn/direct/39cbff5e4a7542aba5f63680dbf51521.png)

> **线程数：虚拟用户数/并发数**
> 
> **Ramp-Up:性能测试运行的时间，上面的线程数运行完的时间。**
> 
> **循环次数：如果线程数为10，循环次数为2，那么总共就发送了20次请求**

![](https://i-blog.csdnimg.cn/direct/3150c314ab1348d2abb62ff1be9f2eef.png)

如果选择了永远，就必须要配置调度器，否则性能测试就是一个死循环。

如果配置了调度器，配置了持续时间，就会在2s内不断发送请求

### 2.2、HTTP取样器![](https://i-blog.csdnimg.cn/direct/ff51364d1ac54543984404e155bb7237.png)

![](https://i-blog.csdnimg.cn/direct/28eb4d15447843ed9345e996f19db4a3.png)

我们就可以在聚合报告中查看在两秒内一个发送了多少次请求

### 2.3、查看结果树

出现错误时：

![](https://i-blog.csdnimg.cn/direct/5a046e3838204ed1857c9017ba5cf7b8.png)

我们应该重点关注Load time响应时间和Response code状态码

### 2.4、[HTTP请求](https://so.csdn.net/so/search?q=HTTP%E8%AF%B7%E6%B1%82&spm=1001.2101.3001.7020)默认值

![](https://i-blog.csdnimg.cn/direct/58be5705679345119ebaabc5f23e5a63.png)

**如果是在同一个web系统那么他的每个界面的协议、ip、端口号、内容编码（utf-8）都是一样的，因此我们就可以设置HTTP请求默认值，这样就不用每次都填写了。**

### 2.5、HTTP信息头管理器

**列表页要添加请求信息，否则就会报错**

**添加HTTP管理头，只作用在列表页**

![](https://i-blog.csdnimg.cn/direct/e84ea2b86fa04cdfbc2cae543aa6ff83.png)

**这样运行结果正确**

![](https://i-blog.csdnimg.cn/direct/03a82816f3bd4cb2bacff4a31a4adc27.png)

**下图是开发者工具中列表页的User\_token\_header  需要添加这个名称和值 到HTTP信息头管理器才可以请求成功**

![](https://i-blog.csdnimg.cn/direct/7a17d3b8c9af494e9995f1217b8822e9.png)

**因为我们是给了一个固定的值，它会过期，所以我们就必须要使用JSON提取器来解决问题**

### 2.6、JSON提取器

**接口响应成功，通过提取返回值对应字段，可用于其他接口的参数配置**

**我们可以用登录页的data值来配置别表页**

![](https://i-blog.csdnimg.cn/direct/9e88f532f3ba4608a26f3cf421069082.png)

**可以对表达式进行测试，看写的对不对：在查看结果树中，将响应数据的格式改为JSON Path Tester，在JSON Path Expression中输入表达式，可以测试提取表达式是否正确**

![](https://i-blog.csdnimg.cn/direct/fdfd35e327aa4972a0e5e15358479858.png)

**补充知识：如何对JSON进行提取**

![](https://i-blog.csdnimg.cn/direct/537da7ad98874b9f840ebd9272ab0498.png)

> \[
> 
> {
> 
> "postTime" : "2024-04-18 05:20:16" ,
> 
>   "title" : "ddddd" ,
> 
> "blogId" : 13 ,
> 
>   "userId" : 3 ,
> 
> "content" : "# 在这⾥写下⼀篇博客 \\r\\ndddd"
> 
> },
> 
>  {
> 
> "postTime" : "2022-10-22 02:38:21" ,
> 
> "title" : " 同学，请问你今天学习了吗 " ,
> 
> "blogId" : 12 ,
> 
>   "userId" : 3 ,
> 
> "content" : " 今天是 2022 年 10 ⽉ 22 ⽇ 17:42 分，为了能够早⽇将最新版本的测试课件呈现
> 
> 给同学们，我已经开始奋 ..."
> 
> }
> 
> \]
> 
> **获取相应中的所有blogId元素：$..blogId**
> 
> **获取第⼀个blogId元素：$.\[0\]blogId**

**测试提取正确之后，就将值写到JSON提取器中。**

![](https://i-blog.csdnimg.cn/direct/90e6668bd9c540fa803c14a716a54884.png)

**书写格式：${变量名}**

![](https://i-blog.csdnimg.cn/direct/146ac37614144ab7bf3a5312e2572b1f.png)

> **那为什么要添加这个呢？？？**
> 
> HTTP协议本身是无状态的，服务器需要通过会话标识来识别用户身份。
> 
> 用户登录之后，服务器返回一个认证凭证，后续请求必须携带该凭证（如访问列表页），否则服务器会返回401/403未授权
> 
> *   浏览器在登录后会自动管理Cookie/Token，并在后续请求中自动附加这些信息。
>     
> *   JMeter需要手动实现这一过程，否则列表页请求会被视为“未登录用户”的请求。
>     
> 
> 1.  **JMeter如何实现？**
>     
> 
> *   通过 **提取器（正则/JSON）** + **HTTP信息头管理器** 或 **Cookie管理器** 动态传递凭证。
>     

![](https://i-blog.csdnimg.cn/direct/507305cd22474b4988685f30274d4572.png)

**若多个接口中都有符合条件JSON字段，则会发生覆盖**

**要将提取用户凭证（JSON提取器）放在登录的下面，然后只要一个HTTP信息头管理器**

**token只取登录接口返回值里的data字段。然后直接保存在HTTP信息头管理器**

![](https://i-blog.csdnimg.cn/direct/6ffc40fd133f4da8bffae63695e2f9bb.png)

**不能只看通过了、和响应时间、还有状态码没问题就代表没有问题，还要查看响应体，要返回博客的标题和博客的内容**

**当有两百个详情页接口，每个接口都要用到写死的id值，而这个id值后续可能需要修改----最好的方式就是用批量修改的方式**

![](https://i-blog.csdnimg.cn/direct/8af7bcae198e40fe8a26aa3835b0edd3.png)

**补充：**

**为什么postman可以请求成功，但是放到jmeter之后就请求失败了？**

**我们可以将把开发者工具上的数据和jmeter的数据进行对比进行对比。在postman上验证一下是不是这个问题，但是修改的时候要注意作用域问题。**

### **2.7、JSON断言**

![](https://i-blog.csdnimg.cn/direct/f43e24ca782842628b9e7aff976f6fe8.png)

**举例：**

> 1、**检查字段是否存在**
> 
> ![](https://i-blog.csdnimg.cn/direct/f8bf0ab7222b4c7dacfa6faf27a1a61b.png)
> 
> **1）JOSN Path exists：这个值是点击查看结果树，将格式选为JSON Path Tester 然后在输入框中输入JSON提取的书写格式，对JSON进行提取**
> 
> **2）不选中同时验证字段值**
> 
> **3）不选中选使用正则匹配**
> 
> **4）不输入预期值**
> 
> **如果 `userId` 存在，断言通过；否则失败**
> 
> **2、验证字段值**
> 
> ![](https://i-blog.csdnimg.cn/direct/6a80df4de7a4400cb287f5876dd27557.png)
> 
> **1）JOSN Path exists：这个值是点击查看结果树，将格式选为JSON Path Tester 然后在输入框中输入JSON提取的书写格式，对JSON进行提取：$.code**
> 
> **2）选中同时验证字段值**
> 
> **3）不选中使用正则匹配**
> 
> **4）输入预期值：200**
> 
> **如果 `code` 等于 `200`，断言通过；否则失败。**
> 
> **3、使用正则表达式匹配**
> 
> ![](https://i-blog.csdnimg.cn/direct/27455048a59548ad92bb0d603ac61cd9.png)
> 
> **1）JOSN Path exists：这个值是点击查看结果树，将格式选为JSON Path Tester 然后在输入框中输入JSON提取的书写格式，对JSON进行提取：$.email**
> 
> **2）选中同时验证字段值**
> 
> **3）选中使用正则匹配**
> 
> **4）输入预期值：`.+@.+\\..+` （匹配邮箱格式）**
> 
> **如果 `email` 符合正则表达式，断言通过。**

**前后JSON的关系**

![](https://i-blog.csdnimg.cn/direct/0e35dac35cf74b6493fc665832085b96.png)

         **通过变量提取+断言机制**

### 2.8、同步定时器

**我们要实现线程并发执行就必须添加同步定时器**

![](https://i-blog.csdnimg.cn/direct/2257624ad8a04963b79b462ba5f779dc.png)

**如果没有打开循环，那么最好配置和开始设定的线程数相同的数字，大于就会一直等，小于就小于就会导致后面的线程数量达不到就不运行。打开循环之后是可以的。**

![](https://i-blog.csdnimg.cn/direct/24dd479a44ee468ebb2ccba57b062cca.png)

**添加了同步定时器之后，线程是在都准备好之后才开始的，就可以做到并发**

### **2.9、CSV数据文件设置**

**为了模拟更真实的登录环境，我们需要提供更多的用户和密码来实现登录操作**

![](https://i-blog.csdnimg.cn/direct/517736d3f23e49fe8bbaebe85f0a4386.png)

![](https://i-blog.csdnimg.cn/direct/432651464076483890215ac70805a52b.png)

在当前文件的相同文件夹里面添加一个execl表格，里面写用户名和密码。遇到文件结束符再次循环选True，就会循环的去模拟登录。

### 2.10、HTTP Cookie管理器

![](https://i-blog.csdnimg.cn/direct/ccd9420f325646988f41fcb9b15241f9.png)

> **HTTP Cookie管理器像Web浏览器⼀样存储和发送Cookie。如果HTTP请求并且响应包含cookie,则 Cookie管理器会⾃动存储该cookie,并将其⽤于将来对该特定⽹站的所有请求。每个JMeter线程都有 ⾃⼰的"cookie存储区"。因此，正在测试使⽤cookie存储会话信息的⽹站,则每个JMeter线程都将拥 有⾃⼰的会话。此类Cookie不会显⽰在Cookie管理器显⽰屏上,可以使⽤"查看结果树监听器"查看。**
> 
> **缓存配置可选择standard(标准)或compatibility(兼容的),当然也可以⼿⼯添加⼀些cookie.**

![](https://i-blog.csdnimg.cn/direct/a1241958326a4dea9dffae7767de4a63.png)

**每次启动5个线程，隔3s就启动五个线程，这5个线程在1s内启动完成**

![](https://i-blog.csdnimg.cn/direct/fb298e577fc44144a75a27e0f92ad3e8.png)

**让线程持续运行60s，最后每隔1s结束5个线程。**

**补充：查看结果树一般在调试阶段会用到，在运行的时候的一般不用**

**3、测试报告**
----------

**当性能测试完毕之后，我们要出具测试报告**

**打开cmd**

**可以先进入存放当前测试文件的同级目录**

> **输入命令 jmeter -n -t 测试的文件（第一个测试案例.jmx） -l first.jtl -e -o ./first/.(先创建一个文件夹）**

4、性能分析
------

### **通过三大指标来分析性能问题：  
4.1、响应时间**

如果响应时间超过了要求，代表系统到了瓶颈

**注意事项：分析在多少线程的情况下发⽣了超标**

**响应时间的影响因素：**

**1、系统不稳定，有时快又是慢**

**2、随着并发压力变大而慢慢变慢，响应时间变高**

### **4.2、错误率（可靠率）**

**错误率高的原因：**

**1、接⼝请求错误**

**2、服务器⽆法继续处理，达到了瓶颈（代码写的不好，内存泄漏、硬件资源等**

**3、后端系统限流（系统⾥配置了不能超过多少并发）**

### 4.3、吞吐量

**吞吐量越大，性能越好；吞吐量相对稳定或者变低，可能系统达到了性能瓶颈**

**吞吐量变化规律：**

**波动很大：代表系统不稳定**

**慢慢变高再趋于稳定：和并发量强相关。如果并发量小于吞吐量，慢慢增大并发量，吞吐量也会随之增加**

**慢慢变低，并发量也减少了：说明性能测试要结束了，并发减少；也可能是系统变得卡顿，从而导致响应时间变慢，导致单个线程发起的并发量变少。**

本文转自 <https://blog.csdn.net/weixin_74792326/article/details/149963442>，如有侵权，请联系删除。