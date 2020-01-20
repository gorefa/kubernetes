为了让promethus报警更灵活，可以根据prometheus表达式去自定义报警规则。


### 例子

自定义PrometheusRule,通过打标签去匹配到不同的namespace,按照namespace上定义的组发送报警邮件。
PrometheusRule配置：
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: prometheus-operator-0226-lisai-grafana
  labels:
    app: prometheus-operator
    release: prometheus-operator
  namespace: monitoring
spec:
  groups:
  - name: kubernetes-0226-lisai
    rules:
    - alert: grafana mem
      annotations:
        message: grafana mem big
        runbook_url: https://github.com/kubernetes-monitoring/kubernetes-mixin/tree/master/runbook.md#alert-name-kubepersistentvolumeusagecritical
      expr: |-
        sum(container_memory_usage_bytes{namespace="monitoring",pod_name=~"grafana.*"}) / 1024^3 > .01
      for: 1m
      labels:
        namespace: monitoring

```

根据namespace: monitoring匹配alertmanager配置文件中的收件人。  alertmanager配置文件在k8s中是secrets。

alertmanager部分配置：
```yaml
receivers:
- email_configs:
  - to: 1299310393@qq.com
  name: monitoring
- email_configs:
  - to: 15100499406@163.com
  name: default-receiver
- email_configs:
  - to: 297022664@qq.com
  name: loki
route:
  group_by:
  - job
  group_interval: 5m
  group_wait: 30s
  receiver: default-receiver
  repeat_interval: 2h
  routes:
  - match:
      namespace: monitoring
    receiver: monitoring
  - match:
      namespace: loki
    receiver: loki

```

此例子，当增加PrometheusRule时，会根据标签namespace: monitoring 去修改alertmanager的Secret: 1) 增加monitoring收件组 2)增加match关系
