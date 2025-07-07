# 1、Splunk查询日志技巧

我们讨论的是在Splunk中查询日志的技巧，特别是针对API相关日志的快速查询和问题定位。以下是一些关键点：
1. **基本搜索技巧**：
   - 使用关键词搜索：直接在搜索栏输入关键词，如`error`，但这样可能范围太广。
   - 限定时间范围：在时间选择器中选择合适的时间范围，或者使用`earliest`和`latest`命令，例如：
        `earliest=-24h` 表示最近24小时。
   - 指定索引和数据源：如果知道日志所在的索引或源（source），可以指定：
        `index=api_logs source=access.log`
   - 使用字段搜索：如果日志已经提取了字段（如status、method、path等），可以直接使用字段过滤：
        `status=500` 查找所有500错误。
2. **高效查询技巧**：
   - 使用管道符（|）进行逐步过滤：Splunk的搜索语言（SPL）使用管道符连接多个命令，每一步都会缩小结果集。
        `index=api_logs | search status=5* | table _time, status, method, path`
   - 使用`tstats`命令进行快速搜索：当数据量很大时，`tstats`可以利用索引的摘要信息快速搜索，但需要事先定义好数据模型或加速摘要。
        `| tstats count where index=api_logs status>=500 by _time span=1h`
   - 使用`transaction`命令：将相关的多个事件合并为一个事务，例如同一个会话或请求的多个日志事件。
        `index=api_logs | transaction session_id maxevents=10 | search status=500`
   - 使用`stats`命令进行聚合：统计不同状态码的数量，或者按时间窗口统计。
        `index=api_logs | stats count by status, method`
        `index=api_logs | timechart count by status`
3. **针对API日志的查询**：
   - 查找特定API端点：如果日志中有`path`字段，可以搜索特定端点：
        `path="/api/v1/user" | stats count by status`
   - 查找延迟高的请求：如果日志中有响应时间字段（如`response_time`），可以查找响应时间超过阈值的请求：
        `response_time>5000`  # 5秒以上
        `| top limit=10 path by response_time`  # 按路径统计最慢的10个
   - 错误分析：查找非200状态码的请求，并分析原因：
        `status>=400 | stats count by status, path, method`
        `status=500 | table _time, path, method, user_id, error_message`
4. **定位问题**：
   - 关联错误和异常：查找同时出现错误和异常的日志，或者同一个请求中出现的多个错误。
        `index=api_logs (status=500 OR error="*") | transaction request_id | where eventcount>=2`
   - 查看日志详情：通过`reverse`命令将日志按时间倒序排列，便于查看最近发生的错误。
        `index=api_logs status=500 | reverse`
   - 使用`regex`提取更多信息：如果日志中有未提取的字段，可以使用`rex`命令临时提取。
        `| rex field=_raw "error: (?<error_detail>\w+)" | stats count by error_detail`
   - 关联其他日志：如果错误可能由其他系统引起，可以跨索引或跨应用搜索。
        `index=(api_logs OR db_logs) request_id=12345`
5. **保存和分享**：
   - 将常用查询保存为报表或告警，以便重复使用。
   - 创建仪表板，将多个相关查询的结果集中展示。
6. **性能优化**：
   - 避免使用通配符在开头：如`*error`，这样会导致全索引扫描，效率低下。
   - 尽量使用字段过滤，而不是在原始日志中搜索。
   - 使用摘要索引（summary indexing）或数据模型加速查询。
7. **利用Splunk内置功能**：
   - 使用`Pattern`分析功能：Splunk可以自动识别日志中的模式，帮助发现异常模式。
   - 使用`Field Extractor`（字段提取器）提取新字段。
示例查询：
1. 查找最近1小时API日志中500错误的请求，并显示时间、请求方法、路径和用户ID：
   ```
   index=api_logs earliest=-1h status=500 
   | table _time, method, path, user_id
   ```
2. 统计过去24小时每个API端点的错误（4xx和5xx）数量，按错误数降序排列：
   ```
   index=api_logs earliest=-24h (status>=400) 
   | stats count by path, status 
   | sort -count
   ```
3. 查找响应时间最长的10个请求：
   ```
   index=api_logs 
   | top 10 by response_time 
   | table _time, response_time, method, path, user_id
   ```
4. 分析某个特定请求ID的完整日志（跨多个系统）：
   ```
   index=* request_id="ABCDE12345" 
   | sort _time 
   | table _time, index, source, message
   ```
5. 按小时统计API错误率（错误请求数/总请求数）：
   ```
   index=api_logs earliest=-24h 
   | timechart span=1h count as total, count(eval(status>=400)) as errors 
   | eval error_rate=round(errors/total*100,2) 
   | table _time, error_rate
   ```
通过以上技巧和示例，可以快速定位API相关的问题。注意，实际查询需要根据日志的具体格式和字段进行调整。

# 2、Splunk 查询 API 日志及定位问题的核心技巧  
以下整理高效查询 API 日志、定位故障的 Splunk 实践方法，结合 **搜索语法**、**字段提取** 和 **分析策略**：

---

## **1. 快速定位 API 日志的关键命令**
```sql
index=api_logs sourcetype=rest_* 
  (path="/api/v1/user/*" OR endpoint_name="CreateUser") 
  status_code>=400 
  | table _time, client_ip, status_code, latency, request_id, error_message
```
- **关键字段**：  
  `status_code` (HTTP状态码)、`latency` (延迟)、`request_id` (请求唯一ID)、`error_message` (错误信息)。
- **过滤技巧**：  
  `status_code>=400` 聚焦错误日志，`path` 或 `endpoint_name` 指定 API 端点。

---

## **2. 高频问题定位场景**
### **(1) 按错误率分析 API 端点**
```sql
index=api_logs 
| stats count(eval(status_code>=400)) as errors, count as total by endpoint_name 
| eval error_rate=round(errors/total*100, 2) 
| sort - error_rate
```
- 输出：各 API 端点的错误率排序，快速定位故障点。

### **(2) 追踪单个请求全链路日志**
```sql
index=api_logs request_id="REQ-12345" 
| transaction request_id maxspan=10s 
| table _time, service_name, message
```
- 跨服务追踪：通过 `request_id` 关联微服务日志，分析请求全链路。

### **(3) 慢查询分析**
```sql
index=api_logs latency>5000  // 超过5秒的请求
| stats avg(latency) as avg_latency, p95(latency) as p95 by endpoint_name 
| sort - avg_latency
```
- 结合 `p95`/`p99` 分位数识别性能瓶颈。

---

## **3. 智能字段提取技巧**
- **自动字段提取**：  
  点击日志中的键值（如 `status_code=500`），Splunk 自动提取字段。
- **正则提取**：  
  ```sql
  | rex "error=(?<error_code>\d+)"  // 从原始日志提取 error_code 字段
  ```
- **预定义字段**：  
  在 `props.conf` 中配置正则，自动解析 JSON/XML 日志：
  ```ini
  [api_logs]
  EXTRACT-fields = (?i)"status":\s*(?<status_code>\d+)
  ```

---

## **4. 高级分析策略**
### **(1) 错误关联分析**
```sql
index=api_logs status_code=500 
| stats count by error_message, client_ip 
| sort -count
```
- 聚合高频错误信息，识别恶意 IP 或代码缺陷。

### **(2) 版本对比（A/B 测试）**
```sql
index=api_logs endpoint_name="Checkout" 
| stats avg(latency) as latency_rate by api_version 
| where api_version IN ("v2", "v3")
```
- 对比不同 API 版本的延迟/错误率。

### **(3) 依赖服务故障定位**
```sql
index=api_logs error_message="*database timeout*" 
| timechart count span=1h  // 统计数据库超时频率
```
- 识别下游服务（DB、缓存）故障。

---

## **5. 性能优化贴士**
1. **时间范围过滤**：  
   `earliest=-1h` 限定搜索时间窗口，避免全量扫描。
2. **前置过滤**：  
   先使用 `status_code` 等索引字段过滤，再用 `| search` 细化条件。
3. **启用加速摘要**：  
   对高频查询创建 **Data Model** 或 **Summary Index**，提升速度。
4. **避免通配符开头的搜索**：  
   `*error` 改为 `error*` 减少性能开销。

---

# 总结：定位问题的工作流
1. **收敛范围** → 按时间/状态码/端点过滤  
2. **提取字段** → 解析关键字段（错误码、延迟、请求ID）  
3. **聚合分析** → 统计错误率、延迟分位数  
4. **关联追踪** → 通过 `request_id` 追踪全链路  
5. **根因定位** → 分析高频错误模式或性能瓶颈  

> 关键点：善用 `stats`、`timechart`、`transaction` 命令，结合可视化仪表板（如 `| chart`）快速呈现结果。
