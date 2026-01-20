# BI Dashboard → Subagent 使用指南

*将 BI Dashboard 转换为可复用的 gen_sql / gen_report 类型的Subagent*

本指南介绍如何使用 Datus bi_dashboard 功能，将已有 BI 平台中的 Dashboard 资产，自动转换为可调用的 Subagent，用于生成SQL与分析报告。

通过本指南，你将学会：
1.	bi_dashboard 能解决什么问题
2.	bi_dashboard 会生成哪些 Subagent
3.	两种使用入口（CLI 交互 / 命令行工具）
4.	生成后的 Subagent 如何使用
5.	bi_dashboard 的实现位置（便于开发者理解）


## 1. 基础说明
### 1.1 bi_dashboard 是什么？

在真实业务中，BI Dashboard 往往已经沉淀了大量 被验证过的知识：
•	稳定可用的 SQL
•	明确的指标（Metrics）
•	统一的业务口径
•	清晰的分析主题（Subject / Domain）

bi_dashboard 的目标是：

将这些 BI 资产，转换为大模型可以“安全、稳定、可复用”使用的 Subagent。

转换完成后，大模型将不再“自由发挥”，而是 严格在 Dashboard 语义范围内生成 SQL 或分析结论。


### 1.2 会生成哪些 Subagent？

基于一个 BI Dashboard，bi_dashboard 会生成两类 Subagent：

#### 1.2.1 gen_sql Subagent

用途：
•	在既有 Dashboard 语义下生成 SQL
•	遵循已有表结构、指标定义和 SQL 风格

典型使用场景：
•	“在这个 Dashboard 的口径下，帮我写一条新的分析 SQL”
•	“扩展一个新的维度，但不要破坏现有指标定义”


#### 1.2.2 gen_report Subagent

用途：
•	基于 Dashboard 中的指标和语义生成分析结论
•	用于归因分析、业务总结、报告生成

典型使用场景：
•	“总结这个 Dashboard 最近一个月的变化趋势”
•	“分析某个指标波动的可能原因”


## 2. 前置操作

### 2。1 部署Superset
1. 安装 docker 
2. 安装 kubernetes
3. 安装 helm
4. 进入python的数据目录（一般在python安装目录下，比如 /usr/local/python3.12/），执行`sh datus/sample_data/superset/start_superset.sh`


### 2.2 在agent.yml中增加配置
```yaml
agent:
  namespace:
    superset:
      type: postgresql
      host: 127.0.0.1
      port: 15432
      username: superset
      password: superset
      database: examples
  dashboard:
    superset:
      username: admin
      password: admin
      extra:
        provider: db
```


### 2.3 

初始化数据库metadata

```shell
datus-agent bootstrap-kb --namespace superset --kb_update_strategy overwrite
```


## 3. 开始 BI Dashboard → Subagent

### 3.1 入口命令


### 3.1.1 直接使用datus-agent命令
```shell
datus-agent bootstrap-bi --namespace superset
```


#### 3.1.2 在 Datus CLI 中输入：

.bootstrap-bi


### 3.2 交互流程说明

执行命令后，CLI 会引导你完成以下步骤：
1. 选择 BI 平台
  * 例如：Superset / Metabase（取决于已配置的 adaptor，当前只有Superset的实现）
2. 输入 Dashboard URL
  * CLI 会自动解析 Dashboard ID 
3. 确认 Dashboard 信息
4. 选择用于初始化的 Charts
  * 用于 Reference SQL
  * 用于 Metrics / Semantic Model
5. 自动构建 Subagent
  * 抽取 SQL
  * 生成 Semantic Model
  * 初始化 Metrics
  * 创建并保存 Subagent

### 3.3 执行完成后的结果

执行成功后，你将看到类似提示（比如我使用的Dashboard是 Sales Dashboard ）：
```text
Subagent `superset_sales_dashboard` saved.
Subagent `superset_sales_dashboard` bootstrapped.
Attribution Sub-Agent `superset_sales_dashboard_attribution` saved.
Attribution Sub-Agent `superset_sales_dashboard_attribution` bootstrapped.
```

此时，你已经可以在 CLI 中直接使用这些 Subagent。


## 4. 生成后的 Subagent 如何使用？

生成完成后，你可以直接在 CLI 中通过 /subagent_name 调用。



## 5. 什么时候应该使用 bi_dashboard？

✅ 适合的情况
•	已有成熟 BI Dashboard
•	指标口径相对稳定
•	希望让 LLM 严格遵循现有业务语义

❌ 不适合的情况
•	Dashboard 只是临时 Demo
•	SQL / 指标频繁变动
•	尚未形成统一业务口径
