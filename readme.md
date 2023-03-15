helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm show values grafana/loki-stack > loki-stack-values.yaml
helm install loki-stack grafana/loki-stack --values loki-stack-values.yaml -n loki --create-namespace
kubectl get secret --namespace loki loki-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

Uninstall fluentbit:
disable fluentbit in loki-stack-values.yaml, then
helm upgrade --install loki-stack grafana/loki-stack --values loki-stack-values.yaml -n loki --create-namespace

helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install fluentd bitnami/fluentd --set forwarder.enabled=false --set aggregator.enabled=true --set aggregator.configMap=fluentd-output -n loki

Deploy Fluent Operator/fluentbit/fluentd:
git clone https://github.com/fluent/fluent-operator.git
cd fluent-operator

helm repo add fluent https://fluent.github.io/helm-charts
helm repo update
helm upgrade --install fluent-bit fluent/fluent-bit -n loki --values fluentbit-values.yaml

helm install fluent-operator --create-namespace -n fluent fluent/fluent-operator
helm show values fluent/fluent-operator > fluent-operator-override.yaml

Modify fluent-operator-override.yaml
  
  append following content under fluentbit:
    forward:
      enable: true
      host: fluentd.fluent.svc.cluster.localhost
      port: 24224
  enable fluentd and append following content under fluentd:
    loki:
      enable: true
      url: loki-stack.loki.svc.cluster.localhost:3100
      insecure: true
    opensearch:
      enable: false

helm upgrade --install fluent-operator --create-namespace -n fluent fluent/fluent-operator --values fluent-operator-override.yaml




Deploy fluentbit & fluentd:

helm upgrade fluent-operator --create-namespace -n fluent charts/fluent-operator/  --set containerRuntime=containerd,fluentd.enable=true


helm install grafana grafana/grafana --namespace grafana --create-namespace

helm show values grafana/loki > values.yaml

helm upgrade --install --values values.yaml loki grafana/loki -n grafana

helm upgrade --install fluent-bit grafana/fluent-bit --set loki.serviceName=loki-gateway.grafana.svc.cluster.local --set loki.servicePort=80 --set loki.servicePath=/loki/api/v1/push --set config.memBufLimit=50MB