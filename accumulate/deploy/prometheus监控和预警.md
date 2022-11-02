# 预警内容和级别

## 字典项

| 级别 | 代码     | 方式       |
| ---- | -------- | ---------- |
| 致命 | critical | 电话       |
| 严重 | error    | 钉钉or邮件 |
| 警告 | warn     | 钉钉or邮件 |

## 具体项

| 描述                         | 预警级别                       | 持续时间 |
| ---------------------------- | ------------------------------ | -------- |
| 主机监控                     |                                |          |
| 主机存活告警                 | critical                       | 10s      |
| 主机内存使用率告警           | 大于75% warn  大于90% critical | 1m       |
| 主机CPU使用率告警            | 大于75%  大于90%               | 1m       |
| 主机磁盘使用率告警           | 大于75%  大于90%               | 1m       |
| 主机Tcp TimeWait数量过多告警 | 大于75%  大于90%               |          |
| 主机iowait较高               | 大于75%  大于90%               |          |
| 主机磁盘读速率               | 大于50MB/s                     | 2m       |
| 主机磁盘写速率               | 大于50MB/s                     | 2m       |
| 磁盘inode操作                | 小于10%                        | 2m       |
| redis监控                    |                                |          |
| redis节点存活告警            |                                | 10s      |
| redis主节点缺失              |                                | 10s      |
| redis存在多个主节点          |                                | 10s      |
| redis消耗过多的配置最大内存  | 大于75%  大于90%               | 1m       |
| redis拒绝新的连接            |                                | 5s       |
| rabbitmq监控                 |                                |          |
| Rabbitmq节点存活告警         |                                | 10s      |
| rabbitmq占用内存过高         | 大于75%  大于90%               | 2m       |
| rabbitmq过多的连接           | 大于1000                       | 1m       |
| Rabbitmq过多消息堆积         | 大于10000                      | 2m       |
| JVM监控                      |                                |          |
| Tomcat节点存活告警           |                                | 1m       |
| 使用堆内存过多               | 大于80%                        | 2m       |

# 安装

相关配置文件[路径](https://gitee.com/icxx/dev_notes/tree/master/deploy/prometheus)

## prometheus 安装

### 创建目录和配置文件

创建数据映射目录
```shell 
mkdir -p /home/ljdw/prometheus/data  
mkdir -p /home/ljdw/prometheus/rule
```

在目录/home/ljdw/prometheus下创建 prometheus.yml 配置文件

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           - 172.31.2.189:19093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "/etc/prometheus/rule/*.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
      
  - job_name: abroad_node_211
    static_configs:
    - targets: ['172.31.2.189:19100']
      labels:
          instance: abroad_node_211
  - job_name: abroad_node_251
    static_configs:
    - targets: ['172.31.2.180:19100']
      labels:
          instance: abroad_node_251
  - job_name: abroad_node_30
    static_configs:
    - targets: ['172.31.2.181:19100']
      labels:
          instance: abroad_node_30  


  - job_name: abroad_redis
    static_configs:
    - targets: 
      - redis://172.31.2.189:6379
      - redis://172.31.2.180:6379
      - redis://172.31.2.181:6379
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 172.31.2.189:19101
        
  - job_name: abroad_rabbitmq
    static_configs:
    - targets: ['172.31.2.189:19102']
      labels:
          instance: abroad_rabbitmq_8.210.47.211

  - job_name: web_abroad_211
    static_configs:
    - targets: ['172.31.2.189:19500']
      labels:
          instance: ljdw_web_8.210.47.211
          
  - job_name: web_abroad_30
    static_configs:
    - targets: ['172.31.2.181:19500']
      labels:
          instance: ljdw_web_47.52.42.30
          
  - job_name: web_abroad_251
    static_configs:
    - targets: ['172.31.2.180:19500']
      labels:
          instance: ljdw_web_47.75.151.251
```

注意：
- rule_files 配置中，指定目录下的所有.yml 文件都起效。
- 需要根据exporter的ip进行修改配置

### docker 启动命令
```shell
docker run -d  --name prometheus --restart=always -p 19090:9090 -v /home/ljdw/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml -v /home/ljdw/prometheus/data:/prometheus/data -v /home/ljdw/prometheus/rule:/etc/prometheus/rule prom/prometheus

```
说明：

- 镜像内默认存储路径  /prometheus 
- 默认的配置文件路径 配置文件路径
- -p 19090:9090 端口映射
- -v /home/ljdw/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml 挂载本地的配置文件
- -v /home/ljdw/prometheus/data:/prometheus/data 挂载本地的数据目录
- -v /home/ljdw/prometheus/rule:/etc/prometheus/rule 挂载规则文件目录，需要在配置文件中指定

### 验证

访问 http://ip:port

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221027174129.png)
## alertmanager 安装

### 创建目录和配置文件

在目录/home/ljdw/prometheus/alertmanager 创建 alertmanager.yml 配置文件
alertmanager.yml 

```yml
oute:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 120m
  receiver: 'web_hook'
receivers:
- name: 'web_hook'
  webhook_configs:
  - url: 'http://api.aiops.com/alert/api/event/prometheus/73a370ea6deb48a6b8434a00fd22cc12'
#  - url: 'http://api.aiops.com/alert/api/event/prometheus/73a370ea6deb48a6b8434a00fd22cc6a'
    send_resolved: true
  - url: 'http://172.31.2.189:19094/dingtalk/webhook1/send'
    send_resolved: true

```

注意：
- 配置了两个webhook路径，第一个是睿象云平台，第二个是钉钉提醒，需要安装prometheus-webhook-dingtalk

### docker 命令

```shell
docker run -d --name alertmanager --restart=always -p 19093:9093 -v /home/ljdw/prometheus/alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml prom/alertmanager
```

### 验证

访问 http://ip:port

![](https://raw.iqiq.io/huicxx/md-pic-bed/main//images/20221027174647.png)

## 钉钉提醒

参考：https://blog.csdn.net/weixin_61718665/article/details/122475614
模板参考：https://blog.csdn.net/qq_36648860/article/details/115767873

### 创建目录和配置文件

```shell
mkdir -p /home/ljdw/prometheus/dingtalk
```

创建配置文件（config.yml）和模板文件（template.tmpl）

config.yml内容：
```yaml
## Request timeout
# timeout: 5s

## Uncomment following line in order to write template from scratch (be careful!)
#no_builtin_template: true

## Customizable templates path
templates:
   - /etc/prometheus-webhook-dingtalk/template.tmpl
#  - contrib/templates/legacy/template.tmpl

## You can also override default template using `default_message`
## The following example to use the 'legacy' template from v0.3.0
#default_message:
#  title: '{{ template "legacy.title" . }}'
#  text: '{{ template "legacy.content" . }}'

## Targets, previously was known as "profiles"
targets:
  webhook1:
    url: https://oapi.dingtalk.com/robot/send?access_token=a1787d5c555b2fae09b82bca48ffe68a565131789a493d6523a8b9827fffa359
    secret: SECb9c12106ab6d64872df62a7c77243d5c946c4b9c85c8dfe1528d07eae2150942
# Customize template content
    message:
      title: '{{ template "ding.link.title" . }}'
      text: '{{ template "ding.link.content" . }}'

```
template.tmpl 内容：

```shll
{{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
{{ define "__alertmanagerURL" }}{{ .ExternalURL }}/#/alerts?receiver={{ .Receiver }}{{ end }}

{{ define "__text_alert_list" }}{{ range . }}
**Labels**
{{ range .Labels.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}
**Annotations**
{{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}
**Source:** [{{ .GeneratorURL }}]({{ .GeneratorURL }})
{{ end }}{{ end }}

{{ define "default.__text_alert_list" }}{{ range . }}
---
**告警级别:** {{ .Labels.severity | upper }}

**运营团队:** {{ .Labels.team | upper }}

**触发时间:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}

**事件信息:** 
{{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


{{ end }}

**事件标签:**
{{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}{{ end }}
{{ end }}
{{ end }}
{{ define "default.__text_alertresovle_list" }}{{ range . }}
---
**告警级别:** {{ .Labels.severity | upper }}

**运营团队:** {{ .Labels.team | upper }}

**触发时间:** {{ dateInZone "2006.01.02 15:04:05" (.StartsAt) "Asia/Shanghai" }}

**结束时间:** {{ dateInZone "2006.01.02 15:04:05" (.EndsAt) "Asia/Shanghai" }}

**事件信息:**
{{ range .Annotations.SortedPairs }}> - {{ .Name }}: {{ .Value | markdown | html }}


{{ end }}

**事件标签:**
{{ range .Labels.SortedPairs }}{{ if and (ne (.Name) "severity") (ne (.Name) "summary") (ne (.Name) "team") }}> - {{ .Name }}: {{ .Value | markdown | html }}
{{ end }}{{ end }}
{{ end }}
{{ end }}

{{/* Default */}}
{{ define "default.title" }}{{ template "__subject" . }}{{ end }}
{{ define "default.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
{{ if gt (len .Alerts.Firing) 0 -}}

![警报 图标](https://gitee.com/icxx/springcloudlearn/raw/master/images/alarm.jpg)
**====侦测到故障====**
{{ template "default.__text_alert_list" .Alerts.Firing }}


{{- end }}

{{ if gt (len .Alerts.Resolved) 0 -}}
{{ template "default.__text_alertresovle_list" .Alerts.Resolved }}


{{- end }}
{{- end }}

{{/* Legacy */}}
{{ define "legacy.title" }}{{ template "__subject" . }}{{ end }}
{{ define "legacy.content" }}#### \[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}\] **[{{ index .GroupLabels "alertname" }}]({{ template "__alertmanagerURL" . }})**
{{ template "__text_alert_list" .Alerts.Firing }}
{{- end }}

{{/* Following names for compatibility */}}
{{ define "ding.link.title" }}{{ template "default.title" . }}{{ end }}
{{ define "ding.link.content" }}{{ template "default.content" . }}{{ end }}
```

## Grafana 安装（选择安装）

### 创建目录

```shell
mkdir -p /home/ljdw/grafana
```

### docker 启动命令

```shell
docker run -d --name=grafana --restart=always -p 13000:3000   -v /home/ljdw/grafana:/var/lib/grafana  grafana/grafana
```

## 安装prometheus的exporter

官方的exporter插件列表：https://prometheus.io/docs/instrumenting/exporters/

### 主机监控 node_exporter

#### docker安装

```shell
docker run -d --name node-exporter --restart=always -p 19100:9100  -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" prom/node-exporter
```

验证，访问http://localhost:19100/metrics ，查询出指标数据即可


#### 配置规则

```yaml
# 官方exporter地址：https://github.com/prometheus/node_exporter
# 可预警监控信息参考：https://awesome-prometheus-alerts.grep.to/rules#host-and-hardware
# 服务器资源告警策略，labels中内容自定义
groups:
- name: 服务器资源监控
  rules:
    #服务器宕机，持续15秒 
  - alert: 服务器存活告警
    expr: up{job=~"abroad_node.*"} == 0
    for: 10s 
    labels:
      severity: critical
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} 服务器已停止运行!"
      description: "{{ $labels.instance }} 检测到服务器异常停止！" 
    #CPU使用率过高，高于90%，持续1分钟
  - alert: CPU使用率过高(> 90%)
    expr: 100 - ((avg by (instance,job,env)(irate(node_cpu_seconds_total{mode="idle"}[30s]))) *100) > 90
    for: 1m
    labels:
      severity: critical
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} CPU使用率超过90%!"
      description: "{{ $labels.instance }} 检测CPU连续1分钟占用率超出90%！当前值为：{{ $value }}！"
    #内存使用率，高于90%，持续时间为1m
  - alert: 内存使用率过高(> 90%)
    expr: ((node_memory_MemTotal_bytes -(node_memory_MemFree_bytes+node_memory_Buffers_bytes+node_memory_Cached_bytes) )/node_memory_MemTotal_bytes ) * 100 > 90 
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "{{ $labels.instance }} 内存使用率超过90%!"
      description: "{{ $labels.instance }} 检测内存连续1分钟使用率超出90%！当前值为：{{ $value }}！"
    # 磁盘使用率，高于90%，持续1分钟
  - alert: 磁盘使用率 (> 90%)
    expr: 100 - (node_filesystem_free_bytes{fstype=~"ext3|ext4|xfs"} / node_filesystem_size_bytes{fstype=~"ext3|ext4|xfs"} * 100) > 90
    for: 1m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} 挂载分区{{ $labels.device }}使用率超过90%!"
      description: "{{ $labels.instance }} 挂载分区{{ $labels.device }}使用率超出90%！"
    # 磁盘读取速率，高于50M，持续5分钟
  - alert: 磁盘读取速率(> 50 MB/s)
    expr: sum by (instance) (rate(node_disk_read_bytes_total[2m])) / 1024 / 1024 > 50
    for: 2m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} 磁盘读取速率大于50 MB/s。"
      description: "{{ $labels.instance }} 磁盘读取速率大于50 MB/s 连续5分钟，当前值为：{{ $value }}！"  
    # 磁盘写入速率，高于50M  ，持续2分钟
  - alert: 磁盘写入速率(> 50 MB/s)
    expr: sum by (instance) (rate(node_disk_written_bytes_total[2m])) / 1024 / 1024 > 50
    for: 2m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} 磁盘写入速率大于50 MB/s。"
      description: "{{ $labels.instance }} 磁盘写入速率大于50 MB/s 连续2分钟，，当前值为：{{ $value }}！"
    # 磁盘inode操作，小于10%，持续2分钟
  - alert: 磁盘inode操作(< 10% )
    expr: node_filesystem_files_free{mountpoint ="/rootfs"} / node_filesystem_files{mountpoint="/rootfs"} * 100 < 10 and ON (instance, device, mountpoint) node_filesystem_readonly{mountpoint="/rootfs"} == 0
    for: 2m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} 磁盘inode操作小于10%。"
      description: "{{ $labels.instance }} 磁盘inode操作小于10% 连续2分钟，当前值为：{{ $value }}！"


```

#### prometheus 参数采集

在prometheus.yml  添加以下内容（参考），重启Prometheus

```yaml
  - job_name: abroad_node_211
    static_configs:
    - targets: ['172.31.2.189:19100']
      labels:
          instance: abroad_node_211
  - job_name: abroad_node_251
    static_configs:
    - targets: ['172.31.2.180:19100']

```

### redis 监控

exporter的github路径：https://github.com/oliver006/redis_exporter

#### docker安装

```shell 
docker run -d --name redis_exporter --restart=always -p 19101:9121  oliver006/redis_exporter --redis.addr=172.31.2.189:6379 --redis.password=za36987
```

注意：--redis.addr 参数中需要指定本机的ip，不能是127.0.0.1

验证，访问http://localhost:19101/metrics ，查询出指标数据即可

#### 配置规则

规程参考：https://awesome-prometheus-alerts.grep.to/rules#redis

新建redis_rules.yml 文件，放入到 /home/ljdw/prometheus/rule 目录中，重启prometherus。
redis_rules.yml 文件内容如下：


```yaml
# 第三方exporter：https://github.com/oliver006/redis_exporter 
# 可预警监控信息参考：https://awesome-prometheus-alerts.grep.to/rules#redis
# redis告警策略，labels中内容自定义
groups:
- name: redis预警监控
  rules:
    # redis节点宕机
  - alert: redis节点存活告警
    expr: redis_up{instance=~"redis.*"} == 0
    for: 10s
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} redis已停止运行，job 为：{{ $labels.job }}!"
      description: "{{ $labels.instance }} redis检测到已停止运行!！，job 为：{{ $labels.job }}!"
    # redis没有主节点
  - alert: redis主节点缺失
    expr: redis_instance_info{role="master"} < 1
    for: 10s
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} redis主节点缺失，job 为：{{ $labels.job }}!"
      description: "{{ $labels.instance }} redis主节点缺失，job 为：{{ $labels.job }}!"
    # 存在多个主节点
  - alert: redis存在多个主节点
    expr: redis_instance_info{role="master"} > 1
    for: 5s
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} redis存在多个主节点，job 为：{{ $labels.job }}!"
      description: "{{ $labels.instance }} redis存在多个主节点，job 为：{{ $labels.job }}! 当前值为：{{ $value }}！"
    # 消耗配置的最大内存的90%，持续2分钟
  - alert: redis消耗过多的配置最大内存 (> 90%)
    expr: redis_memory_used_bytes / redis_memory_max_bytes * 100 > 90
    for: 1m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} redis中消耗过多的内存（相对于配置中的最大内存），job 为：{{ $labels.job }} !"
      description: "{{ $labels.instance }} redis中消耗过多的内存（相对于配置中的最大内存），job 为：{{ $labels.job }} !当前值为：{{ $value }}！"
    # 拒绝新的连接
  - alert: redis拒绝新的连接
    expr: increase(redis_rejected_connections_total[1m]) > 0
    for: 5s
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} redis拒绝新的连接，job 为：{{ $labels.job }} !"
      description: "{{ $labels.instance }} redis拒绝新的连接，job 为：{{ $labels.job }} !当前值为：{{ $value }}！"

```

#### prometheus 参数采集

在prometheus.yml  添加以下内容，支持集群，仅需安装一个节点即可（参考），重启Prometheus

```yaml
- job_name: abroad_redis
    static_configs:
    - targets: 
      - redis://172.31.2.189:6379
      - redis://172.31.2.180:6379
      - redis://172.31.2.181:6379
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 172.31.2.189:19101

```

### rabbitmq监控

exporter的github路径：https://github.com/kbudde/rabbitmq_exporter

#### docker安装

编写配置文件config.json，内容如下：

```json
{
    "rabbit_url": "http://172.31.2.189:15672",
    "rabbit_user": "ljdw",
    "rabbit_pass": "ljdw123456",
    "publish_port": "9419",
    "publish_addr": "",
    "output_format": "TTY",
    "ca_file": "ca.pem",
    "cert_file": "client-cert.pem",
    "key_file": "client-key.pem",
    "insecure_skip_verify": false,
    "exlude_metrics": [],
    "include_queues": ".*",
    "skip_queues": "^$",
    "skip_vhost": "^$",
    "include_vhost": ".*",
    "rabbit_capabilities": "no_sort,bert",
    "enabled_exporters": [
            "exchange",
            "node",
            "overview",
            "queue"
    ],
    "timeout": 30,
    "max_queues": 0
}

```

docker 启动命令

```shell 
docker run -d --name rabbitmq-export --restart=always  -v /home/ljdw/prometheus/rabbit-export:/etc/rabbitmq -p 19102:9419 kbudde/rabbitmq-exporter -config-file /etc/rabbitmq/config.json
```
注意：--redis.addr 参数中需要指定本机的ip，不能是127.0.0.1

验证，访问http://localhost:19102/metrics ，查询出指标数据即可

#### 配置规则

规程参考：https://awesome-prometheus-alerts.grep.to/rules#redis

新建rabbitmq_rules.yml 文件，放入到 /home/ljdw/prometheus/rule 目录中，重启prometherus。

rabbitmq_rules.yml 文件内容如下：

```yaml
# 第三方exporter：https://github.com/kbudde/rabbitmq_exporter ，有官方版本，但是官方版本有版本限制，暂不适用。官方版本：https://github.com/rabbitmq/rabbitmq-prometheus
# 可预警监控信息参考：https://awesome-prometheus-alerts.grep.to/rules#rabbitmq，有指定队列的监控，目前考虑通用的，暂不做配置
# rabbitmq告警策略，labels中内容自定义
groups:
- name: rabbitmq预警监控
  rules:
    # 节点宕机
  - alert: Rabbitmq节点存活告警
    expr: rabbitmq_up == 0
    for: 10s
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} Rabbitmq节点已停止运行!，job 为：{{ $labels.job }}!"
      description: "{{ $labels.instance }} Rabbitmq节点已停止运行!，job 为：{{ $labels.job }}!"
    # 占用内存过高，持续2分钟，消耗内存大于90%
  - alert: rabbitmq占用内存过高(> 90%)
    expr: rabbitmq_node_mem_used / rabbitmq_node_mem_limit * 100 > 90
    for: 2m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} rabbitmq中消耗过多的内存（相对于配置中的最大内存），job 为：{{ $labels.job }}!"
      description: "{{ $labels.instance }} rabbitmq中消耗过多的内存（相对于配置中的最大内存），job 为：{{ $labels.job }}!当前值为：{{ $value }}！"
    # 过多的连接，持续2分钟，连接数大于1000
  - alert: rabbitmq过多的连接(> 1000)
    expr: rabbitmq_connections > 1000
    for: 1m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} rabbitmq中存在过多的连接，job 为：{{ $labels.job }}!"
      description: "{{ $labels.instance }} rabbitmq中存在过多的连接，job 为：{{ $labels.job }}!当前值为：{{ $value }}"
    # 过多的消息堆积，持续2分钟，堆积数量大于10000
  - alert: rabbitmq 过多的消息堆积(> 50000)
    expr: rabbitmq_queue_messages_ready_global > 50000
    for: 2m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} rabbitmq过多的消息堆积!"
      description: "{{ $labels.instance }} rabbitmq过多的消息堆积，job 为：{{ $labels.job }}!当前值为：{{ $value }}"

```

#### prometheus 参数采集

在prometheus.yml  添加以下内容（参考），重启Prometheus

```yaml
  - job_name: abroad_rabbitmq
    static_configs:
    - targets: ['172.31.2.189:19102']
      labels:
          instance: abroad_rabbitmq

```


### tomcat 监控

exporter的github路径：https://github.com/prometheus/jmx_exporter

#### 安装

需要下载 jmx_prometheus_javaagent-0.17.1.jar ，地址：https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.17.2/jmx_prometheus_javaagent-0.17.2.jar

创建tomcat.yml 文件，内容如下：

```yaml
lowercaseOutputLabelNames: true
lowercaseOutputName: true
rules:
- pattern: 'Catalina<type=GlobalRequestProcessor, name=\"(\w+-\w+)-(\d+)\"><>(\w+):'
  name: tomcat_$3_total
  labels:
    port: "$2"
    protocol: "$1"
  help: Tomcat global $3
  type: COUNTER
- pattern: 'Catalina<j2eeType=Servlet, WebModule=//([-a-zA-Z0-9+&@#/%?=~_|!:.,;]*[-a-zA-Z0-9+&@#/%=~_|]), name=([-a-zA-Z0-9+/$%~_-|!.]*), J2EEApplication=none, J2EEServer=none><>(requestCount|maxTime|processingTime|errorCount):'
  name: tomcat_servlet_$3_total
  labels:
    module: "$1"
    servlet: "$2"
  help: Tomcat servlet $3 total
  type: COUNTER
- pattern: 'Catalina<type=ThreadPool, name="(\w+-\w+)-(\d+)"><>(currentThreadCount|currentThreadsBusy|keepAliveCount|pollerThreadCount|connectionCount):'
  name: tomcat_threadpool_$3
  labels:
    port: "$2"
    protocol: "$1"
  help: Tomcat threadpool $3
  type: GAUGE
- pattern: 'Catalina<type=Manager, host=([-a-zA-Z0-9+&@#/%?=~_|!:.,;]*[-a-zA-Z0-9+&@#/%=~_|]), context=([-a-zA-Z0-9+/$%~_-|!.]*)><>(processingTime|sessionCounter|rejectedSessions|expiredSessions):'
  name: tomcat_session_$3_total
  labels:
    context: "$2"
    host: "$1"
  help: Tomcat session $3 total
  type: COUNTER

```

在启动tomcat是，在 JAVA_OPTS 中加入 

```shell 
-javaagent:[ jmx_prometheus_javaagent-0.17.1.jar 文件绝对路径]=19500:[tomcat.yml 文件绝对路径]
# 参考如下
-javaagent:/home/ljdw//jmx_prometheus_javaagent-0.17.1.jar=19500:/home/ljdw/tomcat.yml

```

验证，访问http://localhost:19500/metrics ，查询出指标数据即可


#### 配置规则

ljdw_web_rules.yml 文件内容如下：

```yaml
## JVM告警策略，labels中内容自定义。其他参数待自行编写
groups:
- name: ljdw_web预警监控
  rules:
    # 节点存活告警
  - alert: web模块节点存活告警
    expr: up{job=~"web_abroad.*"}==0
    for: 1m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} 节点已停止运行!!"
      description: "{{ $labels.instance }} 节点已停止运行!！"
    # Jvm 堆内存占用过大，持续2分钟， (> 80%)
  - alert: Jvm 堆内存占用过大 (> 80%)
    expr: jvm_memory_bytes_used{area="heap",instance="ljdw_web_abroad"}/jvm_memory_bytes_max{area="heap",instance="ljdw_web_abroad"} *100 > 80
    for: 2m
    labels:
      severity: warning
      team: 立即定位
    annotations:
      summary: "{{ $labels.instance }} Jvm 堆内存占用过大!"
      description: "{{ $labels.instance }} Jvm 堆内存占用过大，当前值为：{{ $value }}！"

```

#### prometheus 参数采集

在prometheus.yml  添加以下内容（参考），重启Prometheus

```yaml
 - job_name: web_abroad_211
    static_configs:
    - targets: ['172.31.2.189:19300']
      labels:
          instance: web (in:172.31.2.189;out:8.210.47.211)

```



