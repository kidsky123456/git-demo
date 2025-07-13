在Jenkins中设置定时任务自动构建项目，主要通过**构建触发器**中的 **`Build periodically`（定期构建）** 或 **`Poll SCM`（轮询源码变更）** 功能实现，两者均基于 **Cron表达式** 配置时间计划。以下是详细步骤和示例：

---

# 一、两种定时构建的区别

| **触发器类型**         | **作用**                                 | **适用场景**                    |
| ---------------------- | ---------------------------------------- | ------------------------------- |
| **Build periodically** | 按固定周期执行构建（无论源码是否有变更） | 每日/每周强制构建（如生成日报） |
| **Poll SCM**           | 定时检查源码仓库变更，有更新时才触发构建 | 代码提交后自动构建（节省资源）  |

---

# 二、Cron表达式格式

Jenkins使用5字段Cron表达式（省略了秒和年），格式如下：

```plaintext
MINUTE HOUR DAY MONTH DAY_OF_WEEK
```

- **字段含义**：
  - `MINUTE`：分钟（0-59）
  - `HOUR`：小时（0-23）
  - `DAY`：日期（1-31）
  - `MONTH`：月份（1-12）
  - `DAY_OF_WEEK`：星期（0-7，0和7均为周日）

- **符号规则**：
  - `*`：匹配任意值（如 `* * * * *` 每分钟）
  - `,`：多值分隔（如 `0 8,12,18 * * *` 每天8点、12点、18点）
  - `-`：范围（如 `0 9-17 * * 1-5` 工作日9点到17点每小时）
  - `/`：步长（如 `H/15 * * * *` 每15分钟）
  - `H`：哈希值（分散负载，如 `H 2 * * *` 每天2:00左右随机时间执行）

> ⚠️ **注意**：  
> Jenkins的Cron使用系统时区，需确保服务器时间正确。

---

# 三、配置步骤

1. **进入Job配置**  
   打开Jenkins → 选择任务 → **Configure（配置）** → **构建触发器** 部分。

2. **选择触发器类型**  

   - **定时构建（不检查变更）**：勾选 **`Build periodically`**  

     ```plaintext
     Schedule示例：0 3 * * *   # 每天凌晨3点构建
     ```

   - **轮询源码变更**：勾选 **`Poll SCM`**  

     ```plaintext
     Schedule示例：*/10 * * * *   # 每10分钟检查一次源码，有变更则构建
     ```

3. **保存并测试**  
   保存配置后，等待计划时间触发或手动执行一次构建验证。

---

# 四、常用示例

| **场景**                     | **Cron表达式**   | **说明**                    |
| ---------------------------- | ---------------- | --------------------------- |
| 每天凌晨2点构建              | `0 2 * * *`      | 固定时间强制构建            |
| 每30分钟检查源码变更         | `H/30 * * * *`   | 源码更新时触发              |
| 工作日上午9点到下午5点每小时 | `0 9-17 * * 1-5` | 仅工作日执行                |
| 每周一凌晨1点                | `0 1 * * 1`      | 注意：`1`=周一（周日=0或7） |
| 每15分钟（分散负载）         | `H/15 * * * *`   | Jenkins自动计算最佳时间点   |

> 💡 **高级技巧**：  
>
> - 多Job联动：若需A任务完成后触发B任务，在B任务的触发器中勾选 **`Build after other projects are built`**，并填写A任务名称。  
> - 避免资源冲突：大量Job时建议使用 `H` 符号分散执行时间（如 `H 8,12 * * *` 替代 `0 8,12 * * *`）。

---



在 Jenkins 的 Groovy 文件中（通常指 Jenkinsfile 或 Pipeline 脚本），可以通过 `triggers` 指令设置定时任务。以下是两种常见方式：

# 方式一：声明式 Pipeline（推荐）

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

# 方式二：脚本式 Pipeline

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

# 关键说明：

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

# 完整示例：带构建步骤的声明式 Pipeline

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

# 调试技巧：

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



# 五、验证与调试

- **查看计划时间**：保存配置后，Jenkins会在页面显示下次触发时间（如 `Next scheduled time: 2025-07-14 02:00:00`）。
- **日志排查**：若未触发，检查 `系统日志` → `系统信息` → `已计划任务` 中的错误信息。
- **强制测试**：手动修改源码或临时缩短Cron间隔（如 `* * * * *`）快速验证。

> 完整Cron语法参考：[Jenkins官方文档](https://www.jenkins.io/doc/book/pipeline/syntax/#cron-syntax)。
