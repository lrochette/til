# NVidia GPU enablement for runtime

How to make a pipleine run on NVidia GPU enabled nodes 

## AWK/EKS

* Create new node group

```
eksctl create nodegroup \
  --cluster onpremlr \
  --region us-east-1 --node-zones us-east-1a \
  --name gpu-nodes \
  --tags "owner=laurent" \
  --nodes 1 \
  --nodes-min 1 \
  --nodes-max 1 \
  --install-nvidia-plugin \
  --node-type p3.2xlarge 
  ```
  
  * Create a new Codefresh runtime environment (assuming existing runner)
  assuming runner is already in namespace "runner"
  assuming agent is named eks_runner
  ```
  kubectl create ns gpu
  codefresh install runtime --runtime-kube-namespace  gpu
  codefresh attach runtime --agent-name eks_runner --agent-kube-namespace runner --runtime-name cf_onprem_eks/gpu --runtime-kube-namespace gpu --restart-agent
  ```
   * EdGetit runtime Environment
  ```
   cf get re cf_onprem_eks/gpu -o yaml > RE.yaml
  ```
   * Add node selector and toleration
 ```
 dockerDaemonScheduler:
  cluster:
    nodeSelector:
      nvidia.com/gpu: "true"
    tolerations:
    - key: "nvidia.com/gpu"
      operator: "Exists"
      effect: "NoSchedule"
 ```
  * Patch Runtime environment
 ```
 cf patch re -f RE.yaml
 ```
    
  
