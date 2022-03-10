# ARM enablement for runtime

How to make a pipeline run on ARM nodes

## AWK/EKS

* Create new node group

```
eksctl utils update-coredns --cluster onpremlr --approve --region us-east-1
eksctl utils update-kube-proxy --cluster onpremlr --approve --region us-east-1
eksctl utils update-aws-node --cluster onpremlr --approve --region us-east-1
```

```
eksctl create nodegroup \
  --cluster onpremlr \
  --tags "owner=laurent" \
  --region us-east-1 \
  --name arm-nodes \
  --node-type m6g.xlarge \
  --nodes 1\
  --nodes-min 1\
  --nodes-max 3
  ```

  * Create a new Codefresh runtime environment (assuming existing runner)
  assuming runner is already in namespace "runner"
  assuming agent is named eks_runner
  ```
  kubectl create ns arm
  codefresh install runtime --runtime-kube-namespace  arm
  codefresh attach runtime --agent-name eks_runner --agent-kube-namespace runner --runtime-name cf_onprem_eks/arm --runtime-kube-namespace arm --restart-agent
  ```

   * Edit runtime Environment
  ```
   cf get re cf_onprem_eks/arm -o yaml > RE.yaml
  ```
   * Add node selector
   ** dind should run on arm
   ** scheduler should run on amd (assuming we reuse a "normal runner" running on X86)
 ```
 runtimeScheduler:
   nodeSelector:
     kubernetes.io/arch: amd64
...
 dockerDaemonScheduler:
  cluster:
   nodeSelector:
      kubernetes.io/arch: arm64
 ```
 
  * Patch Runtime environment
 ```
 cf patch re -f RE.yaml
 ```
