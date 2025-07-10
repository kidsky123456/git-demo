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

## 关键配置说明：
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

## 常用触发条件：
| 触发块      | 描述                     |
|-------------|--------------------------|
| `failure`   | 仅在构建失败时发送       |
| `success`   | 仅在构建成功时发送       |
| `unstable`  | 构建状态不稳定时发送     |
| `always`    | 无论状态如何都发送       |
| `changed`   | 状态与上次不同时发送     |

# 最佳实践建议：
1. 在`post`区块中按状态分组发送
2. 使用HTML格式提升可读性
3. 附加关键文件（日志/测试报告）
4. 添加构建触发者信息：
   ```groovy
   def user = currentBuild.getBuildCauses()[0].userId
   body: "触发用户: ${user}"
   ```

根据实际需求调整收件人、邮件内容和触发条件，建议使用`emailext`插件获得更专业的邮件通知功能。



如果无法使用`emailext`插件，我们可以使用Jenkins内置的基础`mail`功能。以下是仅使用核心功能的解决方案，无需额外插件权限：

# 基础邮件发送方案 (使用内置`mail`步骤)
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
                // 获取构建状态和基本信息
                def buildStatus = currentBuild.result ?: 'SUCCESS'
                def buildSubject = "Jenkins Build ${buildStatus}: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
                def buildUrl = env.BUILD_URL
                def cause = currentBuild.getBuildCauses()[0].shortDescription ?: "Unknown trigger"
                
                // 创建纯文本邮件内容
                def emailBody = """
                |构建状态: ${buildStatus}
                |项目名称: ${env.JOB_NAME}
                |构建编号: #${env.BUILD_NUMBER}
                |构建地址: ${buildUrl}
                |触发原因: ${cause}
                |持续时间: ${currentBuild.durationString.replace(' and counting', '')}
                |
                |=== 最近变更 ===
                ${getRecentChanges()}
                """.stripMargin()
                
                // 根据状态决定收件人
                def recipients = (buildStatus == 'FAILURE') ? 
                    'failure-team@example.com' : 
                    'dev-team@example.com'
                
                // 发送邮件
                mail(
                    to: recipients,
                    subject: buildSubject,
                    body: emailBody
                )
            }
        }
    }
}

// 获取最近代码变更的方法
def getRecentChanges() {
    try {
        def changes = ""
        def changeLogSets = currentBuild.changeSets
        for (set in changeLogSets) {
            for (entry in set.items) {
                changes += "• [${entry.commitId.substring(0,7)}] ${entry.msg} - ${entry.author}\n"
            }
        }
        return changes ?: "No changes detected"
    } catch (ex) {
        return "Unable to retrieve changes: ${ex.message}"
    }
}
```

# 关键功能说明：

1. **基本构建信息**：
   - 包含构建状态、项目名称、构建编号
   - 构建URL可直接点击查看
   - 显示构建持续时间和触发原因

2. **代码变更记录**：
   - 通过`getRecentChanges()`方法获取最近提交
   - 显示提交ID、提交信息和作者
   - 自动处理无变更的情况

3. **智能收件人分配**：
   - 失败时发送给运维团队
   - 成功时发送给开发团队
   - 可根据需要修改邮箱地址

4. **错误处理**：
   - 使用安全操作符(`?:`)处理空值
   - 异常捕获防止脚本中断
   - 自动处理无变更集的情况

# 增强建议（无需插件）：

1. **添加构建日志片段**：
```groovy
def getLastLogLines(lineCount = 20) {
    try {
        def log = currentBuild.rawBuild.getLog(lineCount).join("\n")
        return "=== 最后${lineCount}行日志 ===\n${log}"
    } catch(ex) {
        return "Unable to retrieve log: ${ex.message}"
    }
}

// 在邮件正文中添加：
body: emailBody + "\n\n" + getLastLogLines()
```

2. **添加构建参数信息**：
```groovy
def paramsInfo = params ? 
    "构建参数:\n" + params.collect { "${it.key}=${it.value}" }.join("\n") : 
    "无参数"

// 在邮件正文中添加
```

3. **多状态处理**：
```groovy
post {
    success {
        sendEmail('SUCCESS', 'dev-team@example.com')
    }
    failure {
        sendEmail('FAILURE', 'ops-team@example.com')
    }
    unstable {
        sendEmail('UNSTABLE', 'qa-team@example.com')
    }
}

def sendEmail(status, recipients) {
    // 邮件发送逻辑
}
```

# 配置要求：
1. **Jenkins系统配置**：
   - 进入 *Manage Jenkins > Configure System*
   - 在 **Jenkins Location** 设置系统管理员邮箱
   - 在 **Extended E-mail Notification** 部分配置SMTP服务器（即使不使用插件）

2. **测试邮件配置**：
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Test Email') {
               steps {
                   script {
                       mail(
                           to: 'your-email@example.com',
                           subject: 'Jenkins邮件配置测试',
                           body: '如果收到此邮件，说明Jenkins邮件配置正确'
                       )
                   }
               }
           }
       }
   }
   ```

# 注意事项：
1. 基础`mail`功能只支持纯文本
2. 日志获取功能需要脚本批准（首次运行需管理员审批）
3. 如果遇到权限问题，可使用更简单的日志获取方式：
```groovy
def log = currentBuild.getLog(100).join("\n")
```

这个方案完全使用Jenkins核心功能，不需要任何额外插件权限，同时提供了关键构建信息的通知功能。
