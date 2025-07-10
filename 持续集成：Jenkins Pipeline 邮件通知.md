本文目录如下：

*   一、背景

*   二、使用默认插件 [Mailer Plugin](https://zhida.zhihu.com/search?content_id=244306923&content_type=Article&match_order=1&q=Mailer+Plugin&zhida_source=entity)

*   2.1 检查插件是否安装

*   2.2 插件配置

*   2.3 测试邮件发送

*   2.3 自由风格任务邮件配置

*   2.4 流水线任务邮件配置

*   2.5 邮件通知结果

*   三、使用增强插件 Email Extension Plugin

*   3.1 安装插件

*   3.2 插件配置

*   3.3 使用默认模板

*   3.4 使用自定义的 [groovy 脚本模板](https://zhida.zhihu.com/search?content_id=244306923&content_type=Article&match_order=1&q=groovy+%E8%84%9A%E6%9C%AC%E6%A8%A1%E6%9D%BF&zhida_source=entity)

*   3.5 使用自定义 html 模板

*   四、总结

  

一、背景
----

上次我们讲解了如何离线部署 Jenkins，这次我们要看看在部署完之后，如何将部署结果通过邮件形式发送出来。

Jenkins 文章汇总如下：

*   1、丝滑的打包部署，一套带走

*   2、喝杯咖啡，一键部署完成！（建议收藏）

*   3、喝杯咖啡，一键部署前端项目

*   4、用代码实现流水线部署后端项目，像诗一般优雅

*   5、如果你还不理解 RBAC，看看 Jenkins 如何做到的

*   6、离线部署 Jenkins 填坑指南

二、使用默认插件 Mailer Plugin
----------------------

默认插件 Mailer Plugin 的功能较简单，能满足基本的要求。

### 2.1 检查插件是否安装

Jenkins 自带了一个发送邮件的插件 Mailer Plugin，如果没有安装，可以下载该插件并导入。下图是安装了该插件的结果。

![](https://pic1.zhimg.com/v2-b378966e7d957924bb6eea22c46334d4_r.jpg)

### 2.2 插件配置

安装好插件之后还需要在全局配置中配置邮箱的地址。

`http://<ip>:8082/manage/configure`

如下图所示，配置了 SMTP 服务器，用户默认邮件后缀，发件箱地址和密码，SMTP 端口。另外还可以测试下邮件发送。

![](https://pica.zhimg.com/v2-56107489cf2915fc71bb95f376e45d24_r.jpg)

### 2.3 测试邮件发送

如果能收到测试邮件，则表示配置成功。

![](https://picx.zhimg.com/v2-ee1ad9e121da6a2bec9b96aa782d7f21_r.jpg)

### 2.3 自由风格任务邮件配置

这个插件支持在自由风格项目中配置邮件通知，也可以用在流水线 Pipeline 脚本中。如下图所示，配置在自由风格项目中的配置：

![](https://picx.zhimg.com/v2-c210ff00324f508be930fe41a74c19c7_r.jpg)

这些配置的含义是当构件失败、不稳定、从不稳定变成稳定以及构件造成不良影响时，会发送邮件通知。

因 Pipeline 更灵活且可以定制邮件模板，所以推荐使用 pipeline 的方式。

### 2.4 流水线任务邮件配置

对应的 pipeline 脚本如下：

~~~javascript
pipeline{ agentany tools{ git'Default' } 
         stages{ stage('获取最新代码'){ 
             steps{ script{ echo"获取最新代码" } } } }
post{ always{ echo'构建结束...' } success{ echo'恭喜您，构建成功！！！' 
                                      mailsubject:"'${env.JOB_NAME}[${env.BUILD_NUMBER}]'执行成功", body:""" <divid="content"> <h1>CI报告</h1> <divid="sum2"> <h2>Jenkins运行结果</h2> <ul> <li>jenkins的执行结果:<a>jenkins执行成功</a></li> <li>jenkins的Job名称:<aid="url_1">${env.JOB_NAME}[${env.BUILD_NUMBER}]</a></li> <li>jenkins的URL:<ahref='${env.BUILD_URL}'>${env.BUILD_URL}</a></li> <li>jenkins项目名称:<a>${env.JOB_NAME}</a></li> <li>JobURL:<ahref='${env.BUILD_URL}'>${env.BUILD_URL}</a></li> <li>构建日志：<a href="${BUILD_URL}console">${BUILD_URL}console</a></li> </ul> </div> <divid="sum0"> </div> </div> """, charset:'utf-8', from:'xxxx@xxx.com.cn', mimeType:'text/html', to:"xxx@xxxx.com.cn" //to:"${Recipient}" } failure{ echo'抱歉，构建失败！！！' mailsubject:"'${env.JOB_NAME}[${env.BUILD_NUMBER}]'执行失败", body:""" <divid="content"> <h1>CI报告</h1> <divid="sum2"> <h2>Jenkins运行结果</h2> <ul> <li>jenkins的执行结果:<a>jenkins执行失败</a></li> // 省略 ... } unstable{ echo'该任务已经被标记为不稳定任务....' } changed{ echo'' } } }
~~~



这种方式得把邮件模板写在 pipeline 脚本中，不美观且改起来麻烦，而且如果有多个脚本都包含了这个模板，则调整模板时，需要改动多个脚本，做了很多重复工作。

### 2.5 邮件通知结果

下图是通过部署流水线任务发送的邮件通知。

![](https://picx.zhimg.com/v2-9f0f3fa39005d401077dc27aa2abd3b9_r.jpg)

我们可以安装另外一个比较强大邮件通知插件，来支持读取邮件模板。

三、使用增强插件 Email Extension Plugin
-------------------------------

该插件可以让你引用自己编写的模板，也可以用它自带的模板。

插件的官网：[https://plugins.jenkins.io/email-ext/](https://link.zhihu.com/?target=https%3A//plugins.jenkins.io/email-ext/)

具体用法可以参考官网文档。

### 3.1 安装插件

安装 Email Extension Plugin 插件，如下图所示：

![](https://pic1.zhimg.com/v2-cb8c0a045918d6b168c8feb37e20ce94_r.jpg)

### 3.2 插件配置

需要在全局配置中配置下邮箱服务器、端口、发件箱账号和密码以及用户邮箱地址后缀，如下图所示。

![](https://pic2.zhimg.com/v2-7dd75ac28807cfd9667091ad9efa5667_r.jpg)

### 3.3 使用默认模板

在 pipeline 中使用默认模板即可，文件名：groovy-html.template。引用模板文件的脚本如下：

`body:'''${SCRIPT,template="groovy-html.template"}''',`

#### 3.3.1 pipeline 完整脚本

`pipeline{ agentany stages{ stage('获取最新代码'){ steps{ script{ echo"获取最新代码" } } } } post{ always{ script{ emailext( subject:"'构建通知:${env.JOB_NAME}-Build#${env.BUILD_NUMBER}-${currentBuild.currentResult}'", recipientProviders:[developers(),requestor()], body:'''${SCRIPT,template="groovy-html.template"}''', to:'xxx@xxx.com.cn', mimeType:'text/html' ) } } } }`

*   `emailext` 是 Jenkins Email Extension 插件提供的函数，用于发送电子邮件通知。

*   发送一封主题为“构建通知: \[项目名称\] - Build # \[构建编号\] - \[构建结果\]”的电子邮件。

*   邮件的收件人包括当前项目的开发人员和触发构建的用户。

*   邮件内容是从 `groovy-html.template` 模板文件中读取并渲染的 HTML 内容。

*   邮件的格式是 HTML，可以包含丰富的样式和布局。

*   额外发送给 `xxx@xxx.com.cn`。

#### 3.3.2 邮件通知结果

部署成功的邮件通知结果如下图所示：

![](https://pica.zhimg.com/v2-32444bbcdb45113253624508c6dadbae_r.jpg)

部署失败的邮件通知结果如下图所示：

![](https://picx.zhimg.com/v2-c4eeb79ee6849e13eaffb79edc1b4fe3_r.jpg)

### 3.4 使用自定义的 groovy 脚本模板

按照官网的说明，可以使用自定义的 groovy 脚本模板。

使用自定义脚本（未与此插件一起打包的脚本）需要 Jenkins 管理员的配合。步骤相对简单：

1.  创建 Groovy 脚本模板。脚本名称以该语言的标准扩展名结尾（即`.groovy`）。模板可以任意命名。

2.  让你的 Jenkins 管理员将脚本放在里面`${[JENKINS_HOME](https://zhida.zhihu.com/search?content_id=244306923&content_type=Article&match_order=1&q=JENKINS_HOME&zhida_source=entity)}/email-templates/`。

3.  使用`$SCRIPT`与模板参数相等的令牌，该模板参数等于您的模板文件名，或者另外使用与自定义脚本名称相等的脚本参数。例如，如果模板文件名为`foobar.template`，则电子邮件内容为`${SCRIPT, template="foobar.template"}`。

当然，如果你不对 groovy 脚本不熟，我们还可以使用 html 模板。

### 3.5 使用自定义 html 模板

#### 3.5.1 Pipeline 脚本配置

读取模板的脚本如下：

`body:'''${FILE,path="/home/jenkins/email-template/email.html"}'''`

完整的 pipeline 脚本如下：

`pipeline{ agentany stages{ stage('获取最新代码'){ steps{ script{ echo"获取最新代码" } } } } post{ always{ script{ emailext( subject:"'构建通知:${env.JOB_NAME}-Build#${env.BUILD_NUMBER}-${currentBuild.currentResult}'", body:'''${FILE,path="/home/jenkins/email-template/email.html"}''', to:'xxx@xxx.com.cn', mimeType:'text/html' ) } } } }`

我们还需要添加对应的邮件模板文件。

#### 3.5.2 添加邮件模板文件

![](https://pic1.zhimg.com/v2-4f45ae2a587a6db94337b725bfa6e220_r.jpg)

文件内容如下：

`<!DOCTYPEhtml> <html> <head> <metacharset="UTF-8"> <title>${ENV,var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title> </head> <bodyleftmargin="8"marginwidth="0"topmargin="8"marginheight="4"offset="0"> <tablewidth="95%"cellpadding="0"cellspacing="0" style="font-size:11pt;font-family:Tahoma,Arial,Helvetica,sans-serif"> <tr> <td>(本邮件是程序自动下发，请勿回复！)</td> </tr> <tr> <td><h2> <fontcolor="#0000FF">构建结果-${BUILD_STATUS}</font> </h2></td> </tr> <tr> <td><br/> <b><fontcolor="#0B610B">构建信息</font></b> <htsize="2"width="100%"byte="center"/></td> </tr> <tr> <td> <ul> <li>项目名称&nbsp;:&nbsp;${PROJECT_NAME}</li> <li>构建编号&nbsp;:&nbsp;第${BUILD_NUMBER}</li> <li>触发方式&nbsp;:${CAUSE}</li> <li>构建日志&nbsp;:<ahref="${BUILD_URL}console">${BUILD_URL}console</a></li> <li>构建&nbsp;&nbsp;Url&nbsp;:&nbsp;<ahref="${BUILD_URL}">${BUILD_URL}</a></li> <li>工作目录&nbsp;:&nbsp;<ahref="${PROJECT_URL}workflow-stage">${PROJECT_URL}workflow-stage</a></li> <li>项目&nbsp;&nbsp;Url&nbsp;:&nbsp;<ahref="${PROJECT_URL}">${PROJECT_URL}</a></li> </ul> </td> </tr> <tr> <td><fontcolor="#0B610B">ChangesSinceLast SuccessfulBuild:</font></b> <hrsize="2"width="100%"byte="center"/></td> </tr> <tr> <td> <ul> <li>历史变更记录:<ahref="${PROJECT_URL}changes">${PROJECT_URL}changes</a></li> </ul>${CHANGES_SINCE_LAST_SUCCESS,reverse=true,format="ChangesforBuild#%n:<br/>%c<br/>",showPaths=true,changesFormat="<pre>[%a]<br/>%m</pre>",pathFormat="%p"} </td> </tr> <tr> <td><b><fontcolor="#0B610B">FailedTestResults</font></b> <hrsize="2"width="100%"byte="center"/></td> </tr> <tr> <td><pre style="font-size:11pt;font-family:Tahoma,Aarial,Helvetica,sans-serif">$FAILED_TESTS</pre> <br/> </td> </tr> <!--> <tr> <td><font><fontcolor="#0B610B">构建日志（最后100行）：</font></b> <hrsize="2"width="100%"byte="center"/></td> </tr> <tr> <td><textareacols="80"rows="30"readonly="readonly" style="font-family:CourierNew">${BUILD_LOG,maxLines=100}</textarea> </td> </tr> <---> </table> </body> </html>`

#### 3.5.3 邮件通知结果

邮件通知结果如下图所示：

![](https://pic3.zhimg.com/v2-76828888e01a0d6ace06d83f472567da_r.jpg)

四、总结
----

这篇文章介绍了如何配置和使用邮件插件来实现邮件通知功能。首先，解释了如何检查和配置默认插件 Mailer Plugin，并测试其发送邮件的能力。然后，介绍了增强插件 Email Extension Plugin 的安装和配置，包含如何使用默认和自定义模板发送邮件。对比了这两种插件的使用方法及其适用场景。

确保插件配置正确和测试邮件发送成功是关键步骤。在使用增强插件时，注意正确编写和配置 groovy 脚本和 HTML 模板，以确保邮件通知内容符合预期。

希望这篇文章对你有所启发。

好的，我的讲话完毕，谢谢大家！

参考资料：

[https://plugins.jenkins.io/email-ext/](https://link.zhihu.com/?target=https%3A//plugins.jenkins.io/email-ext/)

[https://www.cnblogs.com/afei654138148/p/14976490.html](https://link.zhihu.com/?target=https%3A//www.cnblogs.com/afei654138148/p/14976490.html)

[https://blog.csdn.net/shm19990131/article/details/107529572](https://link.zhihu.com/?target=https%3A//blog.csdn.net/shm19990131/article/details/107529572)

本文转自 <https://zhuanlan.zhihu.com/p/702793027>，如有侵权，请联系删除。
