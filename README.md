# Monitor your Azure Kubernetes Cluster (AKS) With Prometheus and Grafana

# Prerequisites

- An AKS cluster provisioned and is in Running state
- Azure CLI
- az aks get-credentials -g {resource-group} -n {aks-cluster-name}
- install Helm for your respective OS - https://github.com/helm/helm/releases


# Helm installation.

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
sudo ./get_helm.sh
```

- Prometheus Helm Chart Download

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo list
helm repo update
git clone https://github.com/prometheus-community/helm-charts
```

# values.yaml File edit

```
cd helm-charts/charts/prometheus
vi values.yaml
```

로컬 환경에서의 설치를 위해 values.yaml 파일을 열어서 PV(Persistent Volume) 설정을 false로 변경해줍니다. 
alertmanager, pushgateway, prometheus server에 대해 enabled 값을 false로 변경해줍니다. 
PV, PVC를 설정하지 않고 배포하면 Pending 상태가 되기 때문에 변경이 필요하지만 실제 운영 환경에서는 PV를 설정해주는 것이 권장됩니다.
enabled: false

- 서비스를 브라우저에서 확인하기 위해 ClusterIP를 NodePort or LoadBalancer로 변경합니다. values.yaml 파일의 Service 부분을 찾아 변경.
- AKS에서 사용하기 위해서는 service를 LoadBalancer로 변경합니다.

nodeExporter, server, pushgateway의 service를 변경합니다.

```
service:
...
  loadBalancerIP: ""
  loadbalancerSourceRanges: []
  servicePort: 80
  nodePort: 3100
  sessionAffinity: None
  type: LoadBalancer
  #type: ClusterIP
```
 
# AKS에 Prometheus 및 Grafana 배포
- namespace 생성 및 배포
```
kubectl create namespace monitoring

helm install prometheus prometheus-community/prometheus -f values.yaml --namespace monitoring
```

- Pod와 Service 확인
```
kubectl get pods --namespace monitoring
kubectl get services --namespace monitoring
```
