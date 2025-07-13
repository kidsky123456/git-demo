这个问题很常见，通常不是配置错误，而是 Jenkins 环境或权限问题。以下是排查和解决方案：

# 常见原因及解决方案：

1. **内容安全策略 (CSP) 限制**
   - **现象**：报告生成但无法显示内容（空白页或控制台错误）
   - **解决方案**：
     ```groovy
     // 在Jenkinsfile最前面添加系统属性设置
     properties([
         systemProperties([
             'hudson.model.DirectoryBrowserSupport.CSP': ''
         ])
     ])
     ```
     或者在 Jenkins 管理后台：
     ```
     管理Jenkins → 脚本命令行 → 运行：
     System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "")
     ```

2. **路径大小写敏感问题**
   - **现象**：Linux 服务器上路径大小写敏感
   - **验证方法**：
     ```groovy
     steps {
         sh 'ls -l reports/html/'  // 检查实际路径大小写
         sh 'pwd'                  // 确认工作空间路径
     }
     ```

3. **文件权限问题**
   - **现象**：Jenkins 用户无权访问报告文件
   - **解决方案**：
     ```groovy
     steps {
         sh 'chmod -R 755 reports/'  // 添加执行权限
         publishHTML(...)
     }
     ```

4. **插件兼容性问题**
   - **排查步骤**：
     - 检查插件版本（HTML Publisher 插件 ≥ 1.25）
     - 临时降级 Jenkins 核心版本测试

# 完整调试脚本：

```groovy
stage('Debug Report Path') {
    steps {
        // 1. 确认报告是否存在
        script {
            def reportPath = "${WORKSPACE}/reports/html/index.html"
            if (fileExists(reportPath)) {
                echo "✅ 报告存在: ${reportPath}"
                // 2. 打印文件信息
                sh "ls -l ${reportPath}"
                // 3. 读取文件内容（调试CSP）
                sh "head -n 20 ${reportPath}"
            } else {
                error "❌ 报告不存在: ${reportPath}"
            }
        }
    }
}

stage('Publish HTML Report') {
    steps {
        // 4. 添加权限修复
        sh 'chmod -R 755 reports/'
        
        // 5. 使用绝对路径发布
        publishHTML(
            target: [
                reportDir: "${WORKSPACE}/reports/html",
                reportFiles: 'index.html',
                reportName: 'DEBUG Report',
                keepAll: true
            ]
        )
    }
}
```

# 进阶排查方法：

1. **检查 Jenkins 系统日志**：
   - 访问 `https://<your-jenkins>/log/all`
   - 过滤 `htmlpublisher` 相关日志

2. **测试不同浏览器**：
   - 尝试 Chrome/Firefox/Safari
   - 禁用浏览器扩展程序

3. **网络代理问题**：
   ```groovy
   // 在Jenkins启动参数中添加（管理Jenkins → 系统配置）
   -Dhudson.model.DirectoryBrowserSupport.CSP="default-src 'none'; script-src 'unsafe-inline'; style-src 'unsafe-inline';"
   ```

4. **替代方案（如果仍未解决）**：
   ```groovy
   stage('Archive Report') {
       steps {
           // 直接归档报告
           archiveArtifacts artifacts: 'reports/html/**/*', fingerprint: true
           
           // 创建手动链接
           script {
               def reportUrl = "${BUILD_URL}artifact/reports/html/index.html"
               currentBuild.description = "<a href='${reportUrl}' target='_blank'>查看报告</a>"
           }
       }
   }
   ```

# 最终验证：

成功发布后，您应该能在以下位置看到报告：
1. 构建页面左侧菜单栏（HTML Report 图标）
2. 构建概要中的直接链接
3. 控制台日志末尾会有类似输出：
   ```
   [htmlpublisher] Archiving HTML reports...
   [htmlpublisher] Archiving at BUILD level /var/lib/jenkins/workspace/project/reports/html to /var/lib/jenkins/jobs/project/builds/1/htmlreports/HTML_20Report
   ```

如果问题仍然存在，请检查：
- Jenkins 服务器磁盘空间 (`df -h`)
- 安全插件（如 Matrix Auth）的权限设置
- 使用 Docker agent 测试是否环境相关：
  ```groovy
  agent {
      docker {
          image 'node:18-alpine'
          args '-u root' // 确保有权限
      }
  }
  ```
