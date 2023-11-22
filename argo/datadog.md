# datadog

Add annotations to service to have DD pick up metrics

in apps/rollouts/overlays/dev/kustomization.yaml

```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: dev
resources:
- ../../base
patchesStrategicMerge:
- argo-rollouts-controller-metrics.yaml
```

kustomization.yaml for service
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    ad.datadoghq.com/service.check_names: |-
      ["openmetrics"]
    ad.datadoghq.com/service.init_configs: |-
      [{}]
    ad.datadoghq.com/service.instances: |-
      [
        {
          "cluster_check": "true",
          "openmetrics_endpoint": "http://argo-rollouts-metrics.dev:8090/metrics",
          "namespace": "cf.dev.argo-rollouts",
          "metrics": [".*"]
        }
      ]
  labels:
    app.kubernetes.io/component: server
    app.kubernetes.io/instance: dev-rollouts
    app.kubernetes.io/name: argo-rollouts-metrics
    app.kubernetes.io/part-of: argo-rollouts
    tags.datadoghq.com/env: "dev"
    tags.datadoghq.com/service: "argo-rollouts-metrics"
    tags.datadoghq.com/version: "0.0.1"
  name: argo-rollouts-metrics
spec:
  selector:
    app.kubernetes.io/name: argo-rollouts-metrics
```
