# private-cluster-setup

## overall idea
- cluster run on machine via k3d
- should be usable by devs
  - create a CICD pipeline for devs to deploy their apps to resources in the cluster
- areas of concern and their solutions
  1. networking (ingress+egress, service discovery, external load balancing)
    - NGINX (ingress controller)
    - Calico (container network interface)
      - pod networking
        - assign ips to pods
        - routing between pods and services
      - enforce NetworkPolicies
    - Istio (service mesh (service networking))
      - Kiali (visualization)
    - MetalLB (external load balancing)
  2. monitoring (metrics)
    - Prometheus
      - Grafana (visualization)
      - Alertmanager (waking people up at 3am)
  3. logs
    - Loki (log aggregator)
      - Grafana
  4. security and access control (RBAC)
    - set and configure Roles and Rolebindings
      - RBAC Manager (simplify binding roles to users via a crd)
      - Kubernetes Dashboard (visualization of Roles and Rolebindings)
  5. certificate manager
    - cert-manager (auto issue and renew certs)
  6. storage (dynamic and persistent storage)
    - Rook and Ceph (dynamic provisioning of storage)
        - Ceph dashboard (visualization)
    - Velero (backups and recovery)
  
## steps
1. install homebrew
2. install non-helm dependencies `brew install docker k3s kubernetes-cli helm k9s`
3. [create the cluster (named 'potato') without flannel so that Calico fits](https://k3d.io/v5.0.0/usage/advanced/calico/)
    - `k3d cluster create potato --k3s-arg '--flannel-backend=none@server:*'`
    - check that cluster is as expected and kubectl is pointed towards potato via `k9s`
4. [install calico](https://docs.tigera.io/calico/latest/getting-started/kubernetes/k3s/quickstart#install-calico)
```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.3/manifests/calico.yaml
```
5. add helm repos
```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo add metallb https://metallb.github.io/metallb
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo add kiali https://kiali.org/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
6. install prometheus and grafana 
```
helm install prometheus prometheus-community/prometheus
helm install grafana grafana/grafana
```
7. 