# Install Grafana Loki

```shell
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install loki grafana/loki --values grafana_loki_values.yaml -n loki --create-namespace
```

# Install Grafana

```shell
helm upgrade --install grafana grafana/grafana -n loki
```
Open Grafana UI and set the datasource url to http://loki-gateway:80, and set the X-Scope-OrgID header to 1.

# Install Fluentd

```shell
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
kubectl apply -f fluentd-output-configmap.yaml
helm upgrade --install fluentd bitnami/fluentd --set forwarder.enabled=false --set aggregator.configMap=fluentd-output -n loki
```

# Install Fluent-bit

```shell
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
helm upgrade --install fluent-bit fluent/fluent-bit -n loki --values fluentbit-values.yaml
```