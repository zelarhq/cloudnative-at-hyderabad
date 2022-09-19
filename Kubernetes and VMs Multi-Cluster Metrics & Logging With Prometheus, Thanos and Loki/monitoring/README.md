## git-repo
```
git clone git@github.com:zelarhq/poc-thanos-loki.git
cd poc-thanos-loki/monitoring
```

## thanos installation
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install thanos bitnami/thanos -f thanos/thanos-values.yaml -n monitoring
```
### ingresses
```
kubectl apply -f thanos/query-ingress.yaml -n monitoring

kubectl apply -f thanos/receiver-ingress.yaml -n monitoring
```

## Prometheus installation
### remote write and external label

```
global:
   external_labels:
     cluster: k3s-cluster-1
      
remoteWrite:
 - url: http://<host name of thanos receiver>/api/v1/receive
   headers:
     THANOS-TENANT: <tenant_id name >
```
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus -f prometheus-cluster-monitoring/prometheus-values.yaml -n monitoring
```
```
kubectl create -f prometheus-cluster-monitoring/prometheus-ingress.yaml -n monitoring
```
or
```
kubectl port-forward svc/prometheus-server 3000:80
```

## Grafana
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana -n monitoring -f grafana/grafana-values.yaml
```
```
kubectl create -f grafana/grafana-ingress.yaml -n monitoring
```
## Loki installation
```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install loki grafana/loki-stack -f loki/loki-values.yaml  -n monitoring
```
```
kubectl create -f loki/loki-ingress.yaml -n monitoring
```
## Promtail
#### Modify the Promtail values.yaml with the hostname of Loki ingress in the client section under the config section
```
clients:
    - url: http://<loki host name>/loki/api/v1/push
```
```
 helm install promtail grafana/promtail -f promtail/promtail-values.yaml -n monitoring
```
## Installing and Configuring Blackbox Exporter in Kubernetes for website monitoring
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus-blackbox prometheus-community/prometheus-blackbox-exporter
```
### add your website url inside the  kube-prometheus-stack chart values.yaml
```
additionalScrapeConfigs: 
- job_name: 'blackbox-external-targets'
  metrics_path: /probe
  params:
    module: [http_2xx]
  static_configs:
    - targets:
          - http://yourwebsite.com/
          - https://www.google.com/
  relabel_configs:
    - source_labels: [__address__]
      target_label: __param_target
    - source_labels: [__param_target]
      target_label: instance
    - target_label: __address__
      replacement: prometheus-blackbox-prometheus-blackbox-exporter:9115
```
```
helm install web-monitor prometheus-community/kube-prometheus-stack -f blackbox-wesite-monitoring/website-prometheus-values.yaml -n monitoirng
```
```
kubectl create -f web-monitor-ingress.yaml -n monitoring
```

dashboard
```
website-monitoring >> 7587
```



