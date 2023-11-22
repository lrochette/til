# Installing istio for Gitops

Reference: https://istio.io/latest/docs/setup/install/helm/

## Installation

```
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update


## Get options available
helm show values istio/base
helm show values istio/istiod
helm show values istio/gateway

kubectl create namespace istio-system
helm install istio-base istio/base -n istio-system
helm install istiod istio/istiod -n istio-system --wait

kubectl create namespace istio-ingress
kubectl label namespace istio-ingress istio-injection=enabled
helm install istio-ingress istio/gateway -n istio-ingress --wait
```

I had an erro on security group
based on https://github.com/cncf/demo/issues/144
I removed the `KubernetesCluster` tag from the SG `eksctl-lr5-cluster/ClusterSharedNodeSecurityGroup`

## Testing

```
k create ns istio-test

k apply -n istio-test \
  -f https://raw.githubusercontent.com/istio/istio/release-1.16/samples/httpbin/httpbin.yaml



## Create gateway
kubectl apply -n istio-test -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingress
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "httpbin.example.com"
EOF


kubectl apply -n istio-test -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - "httpbin.example.com"
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /status
    - uri:
        prefix: /delay
    route:
    - destination:
        port:
          number: 8000
        host: httpbin
EOF

```

###
