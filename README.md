# DexIdP with Minikube
The following are the instructions to run minikube locally and to integrate Dex with it.


## Getting Started

### 1. Install Dex using helm
```
helm repo add dex https://charts.dexidp.io
kubectl create ns dex
helm install --generate-name --wait dex/dex --values ./dex/Values.yaml -n dex
```

### 2. Configure the dex ingress
2.1. Configure the ingress tls configuration
```
kubectl -n dex create secret tls minikube-ingress-tls --key certs/server.key --cert certs/server.crt
```

2.2 Create the ingress
```
kubectl -n dex create ingress dex --rule="/dex*=dex:5556,tls=minikube-ingress-tls"
```

### 3. Update Dex settings
3.1. Apply the necessory changes in dex/config.yaml
3.2. Then apply the following:
```
kubectl create secret generic dex \
  --from-file dex/config.yaml \
  --save-config \
  --dry-run=client -o yaml | \
  kubectl apply -f -
```

### 4. Configure Kubectl using kubelogin plugin
```
kubectl oidc-login setup \
  --oidc-issuer-url=https://minikube/dex \
  --oidc-client-id=kubelogin \
  --oidc-client-secret=kubelogin_password \
  --certificate-authority=$(pwd)/certs/ca.crt
```
