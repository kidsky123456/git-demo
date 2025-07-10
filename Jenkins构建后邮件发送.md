在Jenkins中使用Groovy发送邮件可以通过内置的`mail`步骤或更强大的`emailext`插件实现。以下是两种方法的详细代码示例：

# 方法1：使用基础`mail`步骤（无需额外插件）
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // 你的构建步骤
            }
        }
    }
    post {
        always {
            script {
                // 获取构建状态
                def buildStatus = currentBuild.result ?: 'SUCCESS'
                
                mail(
                    to: 'recipient@example.com',
                    subject: "Jenkins Build ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                    |构建状态: ${buildStatus}
                    |项目名称: ${env.JOB_NAME}
                    |构建编号: ${env.BUILD_NUMBER}
                    |构建地址: ${env.BUILD_URL}
                    |触发原因: ${currentBuild.getBuildCauses()[0].shortDescription}
                    """.stripMargin()
                )
            }
        }
    }
}
```

# 方法2：使用`emailext`插件（推荐，功能更强大）
1. **先安装插件**：  
   前往 Jenkins 管理界面 → 插件管理 → 安装 *Email Extension Plugin*

2. **Groovy代码示例**：
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                // 你的构建步骤
            }
        }
    }
    post {
        failure {
            emailext(
                to: 'team@example.com',
                subject: "❌ 构建失败: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                <h2>构建失败通知</h2>
                <p><b>项目:</b> ${env.JOB_NAME}</p>
                <p><b>构建号:</b> <a href="${env.BUILD_URL}">#${env.BUILD_NUMBER}</a></p>
                <p><b>状态:</b> <span style="color:red;">${currentBuild.result}</span></p>
                <p><b>控制台日志:</b> <a href="${env.BUILD_URL}console">查看日志</a></p>
                """,
                mimeType: 'text/html',
                attachLog: true,    // 附加日志文件
                compressLog: true    // 压缩日志
            )
        }
        success {
            emailext(
                to: 'dev@example.com',
                subject: "✅ 构建成功: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """${currentBuild.currentResult}""",
                attachmentsPattern: '**/test-report.html'  // 附加测试报告
            )
        }
    }
}
```

### 关键配置说明：
1. **Jenkins系统配置**：
   - 进入 *Manage Jenkins > Configure System*
   - 设置 SMTP 服务器和默认发件人邮箱
   - 测试邮件配置是否正确

2. **邮件模板增强技巧**：
   ```groovy
   body: """${groovy.text.StreamingTemplateEngine}
        // 使用模板文件
        def template = libraryResource 'email_template.html'
        def engine = new groovy.text.StreamingTemplateEngine()
        engine.createTemplate(template).make(binding).toString()
   """
   ```

3. **动态收件人列表**：
   ```groovy
   to: emailextrecipients([
        [$class: 'CulpritsRecipientProvider'],
        [$class: 'RequesterRecipientProvider']
   ])
   ```

### 常用触发条件：
| 触发块      | 描述                     |
|-------------|--------------------------|
| `failure`   | 仅在构建失败时发送       |
| `success`   | 仅在构建成功时发送       |
| `unstable`  | 构建状态不稳定时发送     |
| `always`    | 无论状态如何都发送       |
| `changed`   | 状态与上次不同时发送     |

### 最佳实践建议：
1. 在`post`区块中按状态分组发送
2. 使用HTML格式提升可读性
3. 附加关键文件（日志/测试报告）
4. 添加构建触发者信息：
   ```groovy
   def user = currentBuild.getBuildCauses()[0].userId
   body: "触发用户: ${user}"
   ```

根据实际需求调整收件人、邮件内容和触发条件，建议使用`emailext`插件获得更专业的邮件通知功能。
