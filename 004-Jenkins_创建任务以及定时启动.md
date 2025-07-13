一、创建任务
======

1、点击New Item
------------

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325171133698-466374574.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325171133698-466374574.png)

2、输入用户，单击Freestyle project，点击OK
-------------------------------

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325171332860-1983705729.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325171332860-1983705729.png)

3、填写构建步骤，因为是安装在linux上的，所以我们选择Execute shell，随意输入一些简单命令，点击apply
-------------------------------------------------------------

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325173746975-813938062.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325173746975-813938062.png)[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325173921487-180634466.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325173921487-180634466.png)

4、一个简单的job就创建成功了
----------------

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325174106749-1229487748.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325174106749-1229487748.png)

5、点击Build Now 执行构建，并查看Console，可以看到shell命令被执行
--------------------------------------------

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325174250147-1909722904.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325174250147-1909722904.png)

二、创建定时任务
========

 1、进入项目，点击Configure 
--------------------

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325174512791-1249924564.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325174512791-1249924564.png)

2、找到构建触发器(Build Triggers)
-------------------------

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325171920523-120240912.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325171920523-120240912.png)

3、构建触发器说明如下
-----------

### 1） Trigger builds remotely (e.g., from scripts)

在Authentication Token中指定TOKEN\_NAME，然后可以通过连接JENKINS\_URL/job/JOBNAME/build?token=TOKEN\_NAME来启动build。

### 2） Build after other projects are built

可以设置多个依赖的jobs，当任意一个依赖的jobs成功后启动此build。  多个依赖的jobs间使用,隔开。

### 3）  Build periodically

隔一段时间build一次，不管版本库代码是否发生变化

如15 2 \* \* \*表示每天凌晨2.15分的时候进行构建

第1列分钟1～59  
第2列小时0～23  
第3列日1～31  
第4列月1～12  
第5列星期0～7(0和7表示星期日)

### 4） Poll SCM

隔一段时间比较一次源代码如果发生变更，那么就build。否则，不进行build，

`每15分钟构建一次：H/15 * * * * 或*/5 * * * *`  
`每天8点构建一次：H 8 * * *`  
`每天8点~17点，两小时构建一次：H 8-17/2 * * *`  
`周一到周五，8点~17点，两小时构建一次：H 8-17/2 * * 1-5`  
`每月1号、15号各构建一次，除12月：H H 1,15 1-11 *`

第1列分钟1～59  
第2列小时0～23  
第3列日1～31  
第4列月1～12  
第5列星期0～7(0和7表示星期日)

 4、修改时区并设置定时构建
--------------

Build periodically 和 Poll SCM 的时间是以美国东部时间为参考，如果我们直接输入 5 12 \* \* \*的定时规则，定时任务是不会在每天的12点5分执行的，我们需要修改jenkins的时区为Asia/Shanghai，具体步骤如下：

#### 打开jenkins图形界面，找到系统管理->脚本命令行

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325181540211-967487744.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325181540211-967487744.png)

#### 在脚本执行命令行中输入：System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325181733428-1468369914.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210325181733428-1468369914.png)

#### 在构建触发器中设置 TZ=Asia/Shanghai，点击应用

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210326093019150-1329543608.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210326093019150-1329543608.png)

 时间到了自动构建

[![](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210326093049298-257267353.png)](https://img2020.cnblogs.com/blog/1610045/202103/1610045-20210326093049298-257267353.png)

*   [一、创建任务](#tid-Gn83c6)
*   [1、点击New Item](#tid-PKKaGM)
*   [2、输入用户，单击Freestyle project，点击OK](#tid-jCReR6)
*   [3、填写构建步骤，因为是安装在linux上的，所以我们选择Execute shell，随意输入一些简单命令，点击apply](#tid-DZ34js)
*   [4、一个简单的job就创建成功了](#tid-s2XpMZ)
*   [5、点击Build Now 执行构建，并查看Console，可以看到shell命令被执行](#tid-DmWXRK)
*   [二、创建定时任务](#tid-C2iTr4)
*   [1、进入项目，点击Configure](#tid-chjief) 
*   [2、找到构建触发器(Build Triggers)](#tid-rfa8nB)
*   [3、构建触发器说明如下](#tid-kxN5ED)
*   [1） Trigger builds remotely (e.g., from scripts)](#tid-X7mAEw)
*   [2） Build after other projects are built](#tid-jGBxZt)
*   [3）  Build periodically](#tid-hy7dtw)
*   [4） Poll SCM](#tid-jwpDQw)
*   [4、修改时区并设置定时构建](#tid-bNAyT6)
*   [打开jenkins图形界面，找到系统管理->脚本命令行](#tid-YREHrf)
*   [在脚本执行命令行中输入：System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')](#tid-jd7KDf)
*   [在构建触发器中设置 TZ=Asia/Shanghai，点击应用](#tid-AxPkET)

  

\_\_EOF\_\_

[![](//pic.cnblogs.com/avatar/1610045/20220525214335.png)](//pic.cnblogs.com/avatar/1610045/20220525214335.png)

*   **本文作者：** [不自在](https://www.cnblogs.com/testlearn)
*   **本文链接：** [https://www.cnblogs.com/testlearn/p/14578453.html](https://www.cnblogs.com/testlearn/p/14578453.html)
*   **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://msg.cnblogs.com/msg/send/testlearn)我。
*   **版权声明：** 本博客所有文章除特别声明外，均采用 [BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0/ "BY-NC-SA") 许可协议。转载请注明出处！
*   **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。

本文转自 <https://www.cnblogs.com/testlearn/p/14578453.html>，如有侵权，请联系删除。
