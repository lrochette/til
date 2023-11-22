# runner

Note for EBS and Kubernestes > 1.23
https://support.codefresh.io/hc/en-us/articles/7510188292636-Volume-provisioning-issues-after-Kubernetes-upgrade-to-1-23-Amazon-EBS-CSI-driver-

## Install runner with helm

cf runner init runner --generate-helm-values-file ./generated_values.yaml --kube-context-name csdp --kube-namespace runner  --set-default-runtime=false  --exec-demo-pipeline=false --name runner-agent -f values.yaml

helm repo add cf-runtime https://h.cfcr.io/codefresh-inc/runtime
helm repo update
helm upg cf-runtime cf-runtime/cf-runtime -f ./generated_values.yaml --create-namespace --namespace runner -f values-2.yaml

### Cleanup

```
cf delete re csdp/runner
cf delete agent runner-agent
helm uninstall cf-runtime -n runner
kubectl delete ns runner
```

## Runner with Karpenter

* Be sure the storage class AZ matches the Karpenter provisioner aka

```
- key: "topology.kubernetes.io/zone"
  operator: In
  values: ["us-east-1f"]
```

* Install runner
```
cf runner init  \
  --name runner3 \
  --kube-context-name lr3 \
  --skip-cluster-test=false \
  --skip-cluster-integration=true \
  --set-default-runtime=false \
  --exec-demo-pipeline=false \
  --kube-namespace runner \
  --storage-class-name runner-ebs \
  --kube-node-selector=topology.kubernetes.io/zone=us-east-1f \
  --build-node-selector=topology.kubernetes.io/zone=us-east-1f \
  --set-value=Storage.Backend=ebs \
  --set-value=Storage.AvailabilityZone=us-east-1f
```
