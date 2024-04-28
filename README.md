## Kubernetes Monitoring using Prometheus + Loki + Grafana

<div align="center">
   <img src="https://github.com/famasboy888/Kubernetes_monitoring_Prometheus_Loki_Grafana/assets/23441168/f60867c4-514c-40c7-aa95-8021521dd46c" title="Prometheus" alt="Prometheus" width="50%" height="50%"/>
</div>


### 1) Create a namespace

```bash
kubectl create ns loki-stack
```

If we run into error like: Deployment violates PodSecurity

```bash
kubectl label ns loki-stack pod-security.kubernetes.io/enforce=privileged
```

### 1) Add repo from official [Artifact Hub](https://artifacthub.io/packages/helm/grafana/loki-stack)

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

I want to use chart version: `2.9.11` because there is a bug in the latest chart.

```bash
helm show values grafana/loki-stack --version 2.9.11 > values.yaml
```

Customize `values.yaml` to your liking:

values.yaml
```bash
loki:
  enabled: true
  isDefault: true
  url: http://{{(include "loki.serviceName" .)}}:{{ .Values.loki.service.port }}
  readinessProbe:
    httpGet:
      path: /ready
      port: http-metrics
    initialDelaySeconds: 45
  livenessProbe:
    httpGet:
      path: /ready
      port: http-metrics
    initialDelaySeconds: 45
  datasource:
    jsonData: "{}"
    uid: ""
  image:
    tag: 2.8.3

promtail:
  enabled: true
  config:
    logLevel: info
    serverPort: 3101
    clients:
      - url: http://{{ .Release.Name }}:3100/loki/api/v1/push
  image:
    tag: 2.8.3

grafana:
  enabled: true
  sidecar:
    datasources:
      label: ""
      labelValue: ""
      enabled: true
      maxLines: 1000
  image:
    tag: latest

prometheus:
  enabled: true
  isDefault: false
  url: http://{{ include "prometheus.fullname" .}}:{{ .Values.prometheus.server.service.servicePort }}{{ .Values.prometheus.server.prefixURL }}
  datasource:
    jsonData: "{}"

proxy:
  http_proxy: ""
  https_proxy: ""
  no_proxy: ""
```

#### _Note: Be sure to check Daemon Set and PersistentVolumeClaims. I had encountered errors on them._
