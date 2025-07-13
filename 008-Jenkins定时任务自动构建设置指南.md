以下是针对早上7点至8点自动构建的优化 Jenkinsfile 配置，使用 Jenkins 的哈希算法（`H`）在指定时间范围内随机分配执行时间：

```groovy
pipeline {
    agent any
    triggers {
        // 在早上7:00-8:00之间随机时间执行（Jenkins自动分配）
        cron('H 7-8 * * *')
    }
    
    environment {
        // 邮件接收人配置（多人用逗号分隔）
        EMAIL_RECIPIENTS = 'qa-team@example.com,dev-leads@example.com,manager@example.com'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/your/project.git'
            }
        }
        
        stage('Run Cucumber Tests') {
            steps {
                // 执行Cucumber测试
                sh 'mvn clean test -Dcucumber.filter.tags="@smoke"'
            }
            post {
                // 测试失败时保存报告
                failure {
                    archiveArtifacts artifacts: 'target/surefire-reports/*.xml, target/cucumber-reports/**/*', allowEmptyArchive: true
                }
            }
        }
        
        stage('Generate Reports') {
            steps {
                // 生成测试报告
                sh 'mvn cucumber:report'
            }
            post {
                always {
                    // 归档HTML报告
                    archiveArtifacts artifacts: 'target/cucumber-reports/**/*.html', allowEmptyArchive: true
                }
            }
        }
    }
    
    post {
        always {
            // 记录构建时间信息
            script {
                def buildTime = new Date(currentBuild.startTimeInMillis)
                def timeRange = buildTime.format("HH:mm", TimeZone.getTimeZone('Asia/Shanghai'))
            }
        }
        
        success {
            script {
                mail to: env.EMAIL_RECIPIENTS,
                     subject: "✅ SUCCESS: 晨间Cucumber测试通过 - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: """
                     | 晨间自动化测试结果 (执行时间: ${timeRange})
                     | ==================================
                     | **项目**: ${env.JOB_NAME}
                     | **构建号**: ${env.BUILD_NUMBER}
                     | **测试状态**: ${currentBuild.currentResult}
                     | **持续时间**: ${currentBuild.durationString.replace(' and counting', '')}
                     | **报告链接**: ${env.BUILD_URL}cucumber-html-reports/
                     | **控制台日志**: ${env.BUILD_URL}console
                     """.stripMargin()
            }
        }
        
        failure {
            script {
                emailext to: env.EMAIL_RECIPIENTS,
                subject: "❌ FAILED: 晨间Cucumber测试失败 - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                | 晨间自动化测试失败 (执行时间: ${timeRange})
                | ==================================
                | **项目**: ${env.JOB_NAME}
                | **构建号**: ${env.BUILD_NUMBER}
                | **失败原因**: ${currentBuild.buildCauses.shortDescription}
                | **控制台日志**: ${env.BUILD_URL}console
                | **失败报告**: ${env.BUILD_URL}artifact/target/cucumber-reports/overview-features.html
                """.stripMargin(),
                attachLog: true,
                attachmentsPattern: 'target/cucumber-reports/**/*.html'
            }
        }
    }
    
    // 可选：添加构建保留策略
    options {
        // 保留最近10次构建
        buildDiscarder(logRotator(numToKeepStr: '10'))
        
        // 设置超时（避免构建卡死）
        timeout(time: 2, unit: 'HOURS')
    }
}
```

# 关键优化说明：

1. **定时任务配置**：

   ```groovy
   cron('H 7-8 * * *')
   ```

   - `H`：Jenkins 的哈希算法，在指定时间范围内随机分配执行时间
   - `7-8`：表示早上7:00到8:59之间（Jenkins cron 的小时范围包含结束点）
   - 每天执行（无星期限制）

2. **执行时间范围**：

   - 实际执行时间在 **7:00 AM - 8:59 AM** 之间
   - Jenkins 会自动分散任务，避免多个项目同时执行造成资源争用
   - 示例可能执行时间：7:23、7:45、8:12 等

3. **邮件通知增强**：

   - 添加具体执行时间显示：`晨间自动化测试结果 (执行时间: ${timeRange})`
   - 优化邮件主题：明确标注"晨间测试"
   - 失败邮件中包含直接访问失败报告的链接

4. **构建稳定性优化**：

   ```groovy
   options {
       buildDiscarder(logRotator(numToKeepStr: '10'))
       timeout(time: 2, unit: 'HOURS')
   }
   ```

   - 限制保留构建数量（避免磁盘空间不足）
   - 设置2小时超时（防止测试卡死）

# 备选时间方案：

1. **精确控制执行时间**：

   ```groovy
   // 方案1：在7:00-7:59随机时间执行
   cron('H 7 * * *')
   
   // 方案2：固定时间点（多个）
   cron('0 7,8 * * *')  // 7:00和8:00整点执行
   ```

2. **工作日限定**：

   ```groovy
   // 仅周一到周五执行
   cron('H 7-8 * * 1-5')
   ```

3. **周末特别执行**：

   ```groovy
   // 周六周日执行
   cron('H 7-8 * * 6-7')
   ```

# 验证定时配置：

1. 在 Jenkins 控制台检查下次执行时间：

   ```
   Jenkins > 项目 > 配置 > 触发器部分
   ```

   系统会显示类似信息：

   ```
   Next scheduled time: Mon Jul 15 07:32:00 CST 2025
   ```

2. 使用 Jenkins Cron 验证工具：

   ```
   Jenkins > 流水线语法 > 片段生成器 > 选择 "properties: Set Job Properties"
   > 添加 "Build periodically" > 输入 cron 表达式 > 生成脚本
   ```

3. 模拟测试表达式：

   ```groovy
   // 临时修改为每分钟执行测试
   triggers {
       cron('* * * * *')
   }
   // 测试完成后恢复原配置
   ```

# 时区重要提示：

如果 Jenkins 服务器位于不同时区，可通过以下方式处理：

1. **修改 Jenkins 系统时区**：

   ```
   系统管理 > 系统配置 > 时区设置
   ```

2. **在 Pipeline 中指定时区**：

   ```groovy
   triggers {
       cron(timezone: 'Asia/Shanghai', spec: 'H 7-8 * * *')
   }
   ```

3. **使用 UTC 时间计算**：

   ```groovy
   // 例如：上海时间7-8点 = UTC时间23:00-00:00（前一天）
   cron('H 23-0 * * *')
   ```

此配置确保您的 Cucumber 测试每天在早上7-8点之间自动执行，并通过邮件通知团队最新测试结果。
