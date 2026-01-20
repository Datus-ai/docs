# 使用商业智能仪表板（BI Dashboard）生成Subagent

*将转换为可复用的 `gen_sql`和`gen_report`类型的子代理*

本指南介绍如何使用 Datus bi_dashboard 功能，将已有 BI 平台中的 Dashboard 资产，自动转换为可调用的 Subagent，用于生成SQL与分析报告。

通过本指南，你将学会：
1.	`bootstrap-bi` 能解决什么问题
2.	`bootstrap-bi` 会生成哪些 Subagent
3.	两种使用入口（CLI 交互 / 命令行工具）
4.	生成后的 Subagent 如何使用


## 1. 基础说明
### 1.1 `bootstrap-bi` 是什么？

在真实业务中，BI Dashboard 往往已经沉淀了大量 被验证过的知识：
•	稳定可用的 SQL
•	明确的指标（Metrics）
•	统一的业务口径
•	清晰的分析主题（Subject / Domain）

`bootstrap-bi` 的目标是：

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

### 2.1 部署Superset
1. 安装 [docker](https://www.docker.com/get-started/) 
   1. MacOS/Windows: 下载安装
   2. Linux:
      1. Debian/Ubuntu: 
      ```shell
      sudo apt install gnome-terminal
      sudo apt-get update
      sudo apt-get install ./docker-desktop-amd64.deb
      ```
2. 安装 [kubernetes](https://kubernetes.io/zh-cn/docs/tasks/tools/)
    1. MacOS:
       1. 因特尔芯片： `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"`
       2. M芯片： `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"`
    2. Linux:
    3. Windows:
       1. curl: `curl.exe -LO "https://dl.k8s.io/release/v1.35.0/bin/windows/amd64/kubectl.exe"`
       2. 使用Chocolatey `choco install kubernetes-cli`
       3. 使用Scoop `scoop install kubectl`
       4. 使用Winget `winget install -e --id Kubernetes.kubectl`
3. 安装 [helm](https://helm.sh/docs/intro/install/):
   1. MacOS `brew install helm`
   2. Linux
      1. Debian/Ubuntu: 
      ```sudo apt-get install curl gpg apt-transport-https --yes
      curl -fsSL https://packages.buildkite.com/helm-linux/helm-debian/gpgkey | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
      echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://packages.buildkite.com/helm-linux/helm-debian/any/ any main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
      sudo apt-get update
      sudo apt-get install helm
      ```
      2. fedora: `sudo dnf install helm`
      3. Snap: `sudo snap install helm --classic`
      4. FreeBSD: `pkg install helm`
   3. windows
      1. 使用Chocolatey `choco install kubernetes-helm` 
      2. 使用Scoop `scoop install helm`
      3. 使用Winget `winget install Helm.Helm`
   > Linux和Mac系统也可以使用脚本安装： 
    ```bash
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
    chmod 700 get_helm.sh
    ./get_helm.sh
    ```
4. 进入python的数据目录（可通过 `python -c "import datus; print(datus.__file__)"` 查找），执行`sh /usr/local/python3.12/datus/sample_data/superset/start_superset.sh`
5. 安装Datus的扩展包
```shell
# postgresql
pip install datus-postgresql

# generate semantic models and metrics
pip install datus-semantic-metricflow
```

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

## 3. 开始 BI Dashboard → Subagent

### 3.1 入口命令


### 3.1.1 直接使用datus-agent命令
```shell
datus-agent bootstrap-bi --namespace superset
```


#### 3.1.2 在 Datus CLI 中输入：

`.bootstrap-bi`


### 3.2 交互流程说明

执行命令后，CLI 会引导你完成以下步骤：
1. 选择 BI 平台
   * 例如：Superset / Metabase（取决于已配置的 adaptor，当前只有Superset的实现）
2. 输入 Dashboard URL
   * CLI 会自动解析 Dashboard ID 
3. 确认 Dashboard 信息
4. 选择用于初始化的 Charts和表
   * 用于 Reference SQL
   * 用于 Metrics / Semantic Model
5. 自动构建 Subagent
   * 构建 `Metadata`
   * 抽取 `Reference SQL`
   * 生成 `Semantic Model`
   * 初始化 `Metrics`
   * 创建并保存 `Subagent`

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

生成完成后，你可以直接在 CLI 中通过 `/superset_sales_dashboard` 和 `/superset_sales_dashboard_attribution` 调用。具体可参考[子代理介绍](../subagent/introduction.zh.md)


## 5. 什么时候应该使用 bi_dashboard？

✅ 适合的情况
•	已有成熟 BI Dashboard
•	指标口径相对稳定
•	希望让 LLM 严格遵循现有业务语义

❌ 不适合的情况
•	Dashboard 只是临时 Demo
•	SQL / 指标频繁变动
•	尚未形成统一业务口径
