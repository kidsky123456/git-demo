

在 Jenkins 的 Groovy 文件中（通常指 Jenkinsfile 或 Pipeline 脚本），可以通过 `triggers` 指令设置定时任务。以下是两种常见方式：

### 方式一：声明式 Pipeline（推荐）

```groovy
pipeline {
    agent any
    triggers {
        // 方式1：定时构建（无论代码是否有变更）
        cron('H 2 * * *') // 每天凌晨2点左右执行
        
        // 方式2：轮询SCM（检查代码变更）
        pollSCM('H/15 * * * *') // 每15分钟检查一次代码变更
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building project...'
                // 添加你的构建步骤
            }
        }
    }
}
```

### 方式二：脚本式 Pipeline

```groovy
properties([
    pipelineTriggers([
        // 定时构建触发器
        cron('0 9 * * 1-5') // 工作日每天上午9点执行
    ])
])

node {
    stage('Build') {
        echo 'Starting build...'
        // 添加你的构建步骤
    }
}
```

### 关键说明：

1. **语法格式**：

   - `cron()`：用于固定时间构建
   - `pollSCM()`：用于检查代码变更后构建
   - 表达式格式：`'MINUTE HOUR DAY MONTH DAY_OF_WEEK'`

2. **常用表达式示例**：

   ```groovy
   triggers {
       cron('0 12 * * *')    // 每天中午12点
       cron('H 8 * * 1-5')   // 工作日早上8点左右（H表示随机分散）
       pollSCM('*/10 * * * *') // 每10分钟检查代码
   }
   ```

3. **特殊符号**：

   - `H`：哈希值（推荐使用），让 Jenkins 自动分配执行时间
   - `*/n`：每隔 n 个单位执行一次
   - `1-5`：范围（周一至周五）

4. **重要注意事项**：

   - **首次激活**：修改后需手动运行一次 Pipeline 才会激活定时器

   - **时区问题**：基于 Jenkins 服务器时区

   - **查看计划**：在 Pipeline 的 "查看配置" 页面可验证下次执行时间

   - **多触发器**：可同时使用多个触发器

     ```groovy
     triggers {
         cron('H 2 * * *')
         pollSCM('H/30 * * * *')
     }
     ```

### 完整示例：带构建步骤的声明式 Pipeline

```groovy
pipeline {
    agent any
    triggers {
        cron('H 3 * * *') // 每天凌晨3点左右构建
    }
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/your/project.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package' // Maven项目示例
                // 或 sh './gradlew build' // Gradle项目
            }
        }
        
        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
    
    post {
        success {
            mail to: 'team@example.com',
                 subject: "Build Successful: ${currentBuild.fullDisplayName}",
                 body: "Project built successfully at ${env.BUILD_URL}"
        }
    }
}
```

### 调试技巧：

1. 临时使用高频表达式测试（如 `*/2 * * * *` 每2分钟）

2. 在 Jenkins 控制台查看计划任务：

   ```
   Jenkins -> 系统管理 -> 系统日志 -> 日志级别
   添加 logger：hudson.triggers.Timer 级别设为 ALL
   ```

3. 使用 `currentBuild.rawBuild.getTriggers()` 查看激活的触发器

> ⚠️ **最佳实践**：  
>
> - 生产环境推荐使用 `H` 符号分散负载  
> - 复杂调度可在 Jenkins 界面配置后，使用 "流水线语法" 工具生成 Groovy 代码  
> - 结合 `when` 条件控制构建执行（如仅特定分支触发）

以下是针对 Cucumber 自动化测试的完整 Jenkinsfile 配置，包含 `mvn test` 执行测试和多人邮件通知功能：

```groovy
pipeline {
    agent any
    triggers {
        // 每天凌晨3点执行测试
        cron('H 3 * * *') 
    }
    
    environment {
        // 配置邮件接收人（多人用逗号分隔）
        EMAIL_RECIPIENTS = 'qa-team@example.com,dev-leads@example.com,manager@example.com'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/your/project.git'
            }
        }
        
        stage('Run Tests') {
            steps {
                // 使用mvn test执行Cucumber测试
                sh 'mvn clean test -Dcucumber.filter.tags="@smoke"'
            }
            post {
                // 测试失败时保存报告
                failure {
                    archiveArtifacts artifacts: 'target/**/*.html, target/**/*.json', allowEmptyArchive: true
                }
            }
        }
        
        stage('Generate Report') {
            steps {
                // 生成Cucumber报告（示例命令）
                sh 'mvn cucumber:report'
            }
            post {
                always {
                    // 归档测试报告
                    archiveArtifacts artifacts: 'target/cucumber-reports/**/*', allowEmptyArchive: true
                }
            }
        }
    }
    
    post {
        always {
            // 添加测试结果摘要到邮件
            script {
                def summary = """
                | Cucumber 自动化测试结果 |
                | -----------------------|
                | **构建编号**: ${env.BUILD_NUMBER} |
                | **测试状态**: ${currentBuild.currentResult} |
                | **持续时间**: ${currentBuild.durationString.replace(' and counting', '')} |
                | **报告链接**: ${env.BUILD_URL}cucumber-html-reports/ |
                | **控制台日志**: ${env.BUILD_URL}console |
                """
                
                // 移除Markdown格式中的缩进
                summary = summary.stripMargin().trim()
            }
        }
        
        success {
            script {
                // 成功时发送邮件
                mail to: env.EMAIL_RECIPIENTS,
                     subject: "✅ SUCCESS: Cucumber测试通过 - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: summary
            }
        }
        
        failure {
            script {
                // 失败时发送邮件（包含附件）
                emailext to: env.EMAIL_RECIPIENTS,
                subject: "❌ FAILED: Cucumber测试失败 - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: summary,
                attachLog: true,
                attachmentsPattern: 'target/cucumber-reports/**/*.html'
            }
        }
        
        unstable {
            script {
                // 测试不稳定时发送通知
                mail to: env.EMAIL_RECIPIENTS,
                     subject: "⚠️ UNSTABLE: Cucumber测试不稳定 - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: summary
            }
        }
    }
}
```

### 关键配置说明：

1. **Cucumber测试执行**：

   ```groovy
   sh 'mvn clean test -Dcucumber.filter.tags="@smoke"'
   ```

   - 使用 `mvn test` 执行测试
   - 添加 `-Dcucumber.filter.tags` 参数选择特定标签测试用例

2. **多人邮件通知**：

   ```groovy
   environment {
       EMAIL_RECIPIENTS = 'user1@mail.com,user2@mail.com,user3@mail.com'
   }
   ```

   在邮件部分使用 `to: env.EMAIL_RECIPIENTS` 发送给多人

3. **测试报告处理**：

   - 归档HTML/JSON报告：`archiveArtifacts artifacts: 'target/**/*.html'`
   - 邮件附件添加：`attachmentsPattern: 'target/cucumber-reports/**/*.html'`

4. **邮件内容优化**：

   - 添加测试结果摘要（包含构建信息）
   - 使用状态图标（✅ ❌ ⚠️）增强可读性
   - 包含报告链接和控制台日志链接

5. **不同状态处理**：

   - `success`：测试全部通过
   - `failure`：构建失败（附加日志和报告）
   - `unstable`：测试部分失败（Jenkins中设置测试失败阈值）

### 增强功能建议：

1. **添加Slack通知**：

   ```groovy
   post {
       always {
           slackSend channel: '#qa-alerts',
                    color: currentBuild.currentResult == 'SUCCESS' ? 'good' : 'danger',
                    message: "Cucumber测试 ${currentBuild.currentResult}: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
       }
   }
   ```

2. **测试结果趋势分析**：

   ```groovy
   post {
       always {
           junit '**/target/surefire-reports/*.xml'
           cucumber '**/target/cucumber-reports/*.json'
       }
   }
   ```

3. **分支条件执行**：

   ```groovy
   triggers {
       cron(env.BRANCH_NAME == 'main' ? 'H 3 * * *' : '')
   }
   ```

4. **参数化构建**：

   ```groovy
   parameters {
       choice(name: 'TEST_TAGS', choices: '@smoke\n@regression\n@all', description: '选择测试标签')
   }
   stages {
       stage('Run Tests') {
           steps {
               sh "mvn clean test -Dcucumber.filter.tags=\"${params.TEST_TAGS}\""
           }
       }
   }
   ```

### 邮件预览示例：

**成功邮件**：

```
✅ SUCCESS: Cucumber测试通过 - Project-Daily-Test #42

Cucumber 自动化测试结果 
----------------------- 
**构建编号**: 42 
**测试状态**: SUCCESS 
**持续时间**: 12 min 34 sec 
**报告链接**: https://jenkins.example.com/job/Project-Daily-Test/42/cucumber-html-reports/ 
**控制台日志**: https://jenkins.example.com/job/Project-Daily-Test/42/console
```

**失败邮件**：

```
❌ FAILED: Cucumber测试失败 - Project-Daily-Test #43

Cucumber 自动化测试结果 
----------------------- 
**构建编号**: 43 
**测试状态**: FAILURE 
**持续时间**: 8 min 17 sec 
**报告链接**: https://jenkins.example.com/job/Project-Daily-Test/43/cucumber-html-reports/ 
**控制台日志**: https://jenkins.example.com/job/Project-Daily-Test/43/console

(附件包含详细错误报告)
```

> **最佳实践**：
>
> 1. 安装插件增强功能：
>
>    - Email Extension Plugin：高级邮件配置
>    - Cucumber Reports Plugin：可视化报告
>    - Slack Notification Plugin：即时消息通知
>
> 2. 使用 `timeout` 防止测试卡死：
>
>    ```groovy
>    stage('Run Tests') {
>        steps {
>            timeout(time: 30, unit: 'MINUTES') {
>                sh 'mvn test'
>            }
>        }
>    }
>    ```
>
> 3. 清理旧构建节省空间：
>
>    ```groovy
>    options {
>        buildDiscarder(logRotator(numToKeepStr: '10'))
>    }
>    ```
