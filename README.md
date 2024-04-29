## Kubernetes Monitoring using Prometheus + Loki + Grafana (Part 1)

<div align="center">
   <img src="https://github.com/famasboy888/Kubernetes_monitoring_Prometheus_Loki_Grafana_Part_1/assets/23441168/afc55146-c38a-48e5-aefa-a8c719e69f9e" title="Prometheus" alt="Prometheus" width="50%" height="50%"/>
</div>

### 1) Create a namespace

```bash
kubectl create ns loki-stack
```

If we run into error like: Deployment violates PodSecurity

```bash
kubectl label ns loki-stack pod-security.kubernetes.io/enforce=privileged
```

### 2) Add repo from official [Artifact Hub](https://artifacthub.io/packages/helm/grafana/loki-stack)

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

### 3) Confirm all pods are up and running:
```bash
kubectl get all -n loki-stack
```

output:
```bash
NAME                                               READY   STATUS    RESTARTS   AGE
pod/loki-0                                         1/1     Running   0          63m
pod/loki-alertmanager-0                            1/1     Running   0          59m
pod/loki-grafana-58c659d649-xz6dt                  2/2     Running   0          63m
pod/loki-kube-state-metrics-855cbd4649-m97vm       1/1     Running   0          63m
pod/loki-prometheus-node-exporter-f79w5            1/1     Running   0          63m
pod/loki-prometheus-node-exporter-ghj4v            1/1     Running   0          63m
pod/loki-prometheus-node-exporter-vrkfm            1/1     Running   0          63m
pod/loki-prometheus-pushgateway-6d46b8d9c7-n45k7   1/1     Running   0          63m
pod/loki-prometheus-server-c958f85f9-x7d72         2/2     Running   0          57m
pod/loki-promtail-gqhz9                            1/1     Running   0          63m
pod/loki-promtail-vw99w                            1/1     Running   0          63m
pod/loki-promtail-w94w6                            1/1     Running   0          63m

NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/grafana-ext                     NodePort    10.105.149.90    <none>        80:31488/TCP   20m
service/loki                            ClusterIP   10.104.93.71     <none>        3100/TCP       63m
service/loki-alertmanager               ClusterIP   10.96.60.131     <none>        9093/TCP       63m
service/loki-alertmanager-headless      ClusterIP   None             <none>        9093/TCP       63m
service/loki-grafana                    ClusterIP   10.103.177.136   <none>        80/TCP         63m
service/loki-headless                   ClusterIP   None             <none>        3100/TCP       63m
service/loki-kube-state-metrics         ClusterIP   10.101.95.134    <none>        8080/TCP       63m
service/loki-memberlist                 ClusterIP   None             <none>        7946/TCP       63m
service/loki-prometheus-node-exporter   ClusterIP   10.108.212.244   <none>        9100/TCP       63m
service/loki-prometheus-pushgateway     ClusterIP   10.102.160.163   <none>        9091/TCP       63m
service/loki-prometheus-server          ClusterIP   10.110.95.26     <none>        80/TCP         63m

NAME                                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/loki-prometheus-node-exporter   3         3         3       3            3           <none>          63m
daemonset.apps/loki-promtail                   3         3         3       3            3           <none>          63m

NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/loki-grafana                  1/1     1            1           63m
deployment.apps/loki-kube-state-metrics       1/1     1            1           63m
deployment.apps/loki-prometheus-pushgateway   1/1     1            1           63m
deployment.apps/loki-prometheus-server        1/1     1            1           63m

NAME                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/loki-grafana-58c659d649                  1         1         1       63m
replicaset.apps/loki-kube-state-metrics-855cbd4649       1         1         1       63m
replicaset.apps/loki-prometheus-pushgateway-6d46b8d9c7   1         1         1       63m
replicaset.apps/loki-prometheus-server-c958f85f9         1         1         1       63m

NAME                                 READY   AGE
statefulset.apps/loki                1/1     63m
statefulset.apps/loki-alertmanager   1/1     63m
```

Access Grafana Dashboard by exposing grafana service,
<p align="left">
  <img width="80%" height="80%" src="https://github.com/famasboy888/Kubernetes_monitoring_Prometheus_Loki_Grafana/assets/23441168/5e2e66d1-0318-46bf-861b-cc8763df03e8">
</p>

Confirm connections from Loki + Promptail
<p align="left">
  <img width="80%" height="80%" src="https://github.com/famasboy888/Kubernetes_monitoring_Prometheus_Loki_Grafana/assets/23441168/2b236333-c479-44b6-8999-ef47fc671d10">
</p>

Confirm connections from Prometheus
<p align="left">
  <img width="80%" height="80%" src="https://github.com/famasboy888/Kubernetes_monitoring_Prometheus_Loki_Grafana/assets/23441168/b8348952-ccf5-4210-a96f-3b5bcaa566be">
</p>


