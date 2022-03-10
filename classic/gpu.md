# NVidia GPU enablement for runtime

How to make a pipleine run on NVidia GPU enabled nodes

## AWS/EKS

* Create new node group

| Instance Type	| Software	| EC2	| Total |
| ---------- | ---| --- | --- | ------ |
| p2.xlarge | $0.00 |	$0.90	| $0.90/hr |
| p2.8xlarge | $0.00 |	$7.20	| $7.20/hr |
| p2.16xlarge | $0.00 |	$14.40	| $14.40/hr |
| p3.2xlarge | $0.00 |	$3.06	| $3.06/hr |
| p3.8xlarge | $0.00 |	$12.24	| $12.24/hr |
| p3.16xlarge | $0.00 |	$24.48	| $24.48/hr |
| p3dn.24xlarge | $0.00 |	$31.212	| $31.212/hr |


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
  --node-type p2.xlarge
  ```

  * Create a new Codefresh runtime environment (assuming existing runner)
  assuming runner is already in namespace "runner"
  assuming agent is named eks_runner
  ```
  kubectl create ns gpu
  codefresh install runtime --runtime-kube-namespace  gpu
  codefresh attach runtime --agent-name eks_runner --agent-kube-namespace runner --runtime-name cf_onprem_eks/gpu --runtime-kube-namespace gpu --restart-agent
  ```
   * Edit runtime Environment
  ```
   cf get re cf_onprem_eks/gpu -o yaml > RE.yaml
  ```
   * Add node selector
 ```
 dockerDaemonScheduler:
  cluster:
   nodeSelector:
      eks.amazonaws.com/nodegroup: gpu-nodes
 ```
  * Patch Runtime environment
 ```
 cf patch re -f RE.yaml
 ```
