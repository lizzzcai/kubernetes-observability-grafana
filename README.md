# Kubernetes Observability Stack with Grafana

Demo to deploy basic a Grafana Observability Stack (Grafana, Loki, Tempo and Prometheus) on Kubernetes to provide logs, metrics and traces.

## Component

* Metrics Server
* Grafana
* Prometheus Operator Stack
* Loki 2.4.1
* Promtail 2.4.1
* OpenTelemetry
* Tempo 1.2.1

## Optional

* Istio 1.11.4
* BookInfo Demo

## Note

* The Loki and Tempo in the Demo are Monolithic.
* Resources are set for each component.

## Metrics Server

### Prerequisites

Make sure you have Helm [installed](https://helm.sh/docs/intro/install/).

Add Bitnami’s chart repository to Helm:

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
```

To update the chart repository, run:

```sh
helm repo update
```

### Deploy Metrics Server in the kube-system namespace

```sh
helm upgrade --install -f metrics-server.yaml metrics-server bitnami/metrics-server --namespace kube-system
```

## Prometheus Operator Stack

### Prerequisites

Make sure you have Helm [installed](https://helm.sh/docs/intro/install/).

Add Prometheus’s chart repository to Helm:

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

To update the chart repository, run:

```sh
helm repo update
```

### Deploy Prometheus Operator Stack in a custom namespace
```sh
helm upgrade --install -f prometheus-stack.yaml kube-prometheus-stack --create-namespace --namespace kube-prometheus-stack prometheus-community/kube-prometheus-stack
```

## Loki & Promtail

### Prerequisites

Make sure you have Helm [installed](https://helm.sh/docs/intro/install/).

Add Loki’s chart repository to Helm:

```sh
helm repo add grafana https://grafana.github.io/helm-charts
```

To update the chart repository, run:

```sh
helm repo update
```

### Deploy Loki in a custom namespace

```sh
helm upgrade --install -f loki.yaml loki --create-namespace --namespace=loki grafana/loki
```

### Deploy priorityClass for Promtail

```sh
kubectl apply -f high-priority.yaml
```

### Deploy Promtail in a custom namespace

```sh 
helm upgrade --install -f promtail.yaml promtail --create-namespace --namespace=promtail grafana/promtail
```

## Tempo

### Prerequisites

Make sure you have Helm [installed](https://helm.sh/docs/intro/install/).

Add Tempo's chart repository to Helm:

```sh
helm repo add grafana https://grafana.github.io/helm-charts
```

To update the chart repository, run:

```sh
helm repo update
```

### Deploy Tempo in a custom namespace

```sh
helm upgrade --install -f tempo.yaml tempo --create-namespace --namespace=tempo grafana/tempo
```


## OpenTelemetry

### Deploy OpenTelemetry Collector in a custom namespace

```sh
kubectl apply -f open-telemetry-collector.yaml
```

## Istio

### Prerequisites

```sh
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.11.4 sh -

cd istio-1.11.4
export PATH=$PWD/bin:$PATH
istioctl operator init
```

### Install Istio

```sh
kubectl apply -f istio-operator.yaml
```
### Install demo application

```sh
kubectl create ns bookinfo
kubectl label namespace bookinfo istio-injection=enabled --overwrite
kubectl apply -n bookinfo -f https://raw.githubusercontent.com/istio/istio/release-1.11/samples/bookinfo/platform/kube/bookinfo.yaml
# create gateway
kubectl apply -f bookinfo-gateway.yaml
```

### Access the demo application

```sh
kubectl port-forward svc/istio-ingressgateway -n istio-system  8080:80
```
The bookinfo application will now be available at http://localhost:8080.

## Access the Components

### Access Cluster

```sh
export KUBECONFIG=<your kubeconfig>
```

### Grafana

Grafana isn't exposed on a public IP. However, you can use port-forwarding to access your Grafana dashboard.

Set up port-forwarding using the following command:

```sh
kubectl port-forward svc/kube-prometheus-stack-grafana 8080:80 -n kube-prometheus-stack
```

Your Grafana instance will now be available at http://localhost:8080.

Login with the following credentials:

```sh
# skip update the password
Username: admin
Password: admin
```

### Prometheus

The Prometheus Expression Browser enables queries of the data stored in Prometheus. You can access the Prometheus Expression Browser by run the following command:

```sh
kubectl port-forward svc/kube-prometheus-stack-prometheus 9090 -n kube-prometheus-stack
```

Your Prometheus instance will now be available at http://localhost:9090.

### Alertmanager

The Alertmanager manages alerts being sent from Prometheus and can relay those alerts to services such as email, PagerDuty, or Slack.

To access Prometheus Alertmanager, run the following command:

```sh
kubectl port-forward svc/kube-prometheus-stack-alertmanager 9093 -n kube-prometheus-stack
```

Your Alertmanager instance will now be available at http://localhost:9093.

### Metrics Server

Metrics Server is a scalable, efficient source of container resource metrics.

To view node-level metrics, run the following command:

```sh
kubectl top nodes
```

To view pod-level metrics across all namespaces, run the following command:

```sh
kubectl top pods --all-namespaces
```
