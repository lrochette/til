# Karpenter

## installing Karpenter on Cluster

[Getting started](https://karpenter.sh/v0.18.0/getting-started/getting-started-with-eksctl/)

### Init
```
export KARPENTER_VERSION=v0.18.0
export CLUSTER_NAME="lr4"
export AWS_DEFAULT_REGION="us-east-1"
export AWS_ACCOUNT_ID="$(aws sts get-caller-identity --query Account --output text)"
```

### Create cluster

```
eksctl create cluster -f /Users/lrochette/src/lrochette/til/classic/karpenter/cluster.yaml
export CLUSTER_ENDPOINT="$(aws eks describe-cluster --name ${CLUSTER_NAME} --query "cluster.endpoint" --output text)"
```
### Create the KarpenterNode IAM Role

1. First, create the IAM resources using AWS CloudFormation.

```
TEMPOUT=$(mktemp)

curl -fsSL https://karpenter.sh/"${KARPENTER_VERSION}"/getting-started/getting-started-with-eksctl/cloudformation.yaml  > $TEMPOUT \
&& aws cloudformation deploy \
  --stack-name "Karpenter-${CLUSTER_NAME}" \
  --template-file "${TEMPOUT}" \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides "ClusterName=${CLUSTER_NAME}"
```

2. Second, grant access to instances using the profile to connect to the cluster. This command adds the Karpenter node role to your aws-auth configmap, allowing nodes with this role to connect to the cluster.

```
eksctl create iamidentitymapping \
  --username system:node:{{EC2PrivateDNSName}} \
  --cluster "${CLUSTER_NAME}" \
  --arn "arn:aws:iam::${AWS_ACCOUNT_ID}:role/KarpenterNodeRole-${CLUSTER_NAME}" \
  --group system:bootstrappers \
  --group system:nodes
```

### Create the KarpenterController IAM Role

```
eksctl create iamserviceaccount \
  --cluster "${CLUSTER_NAME}" --name karpenter --namespace karpenter \
  --role-name "${CLUSTER_NAME}-karpenter" \
  --attach-policy-arn "arn:aws:iam::${AWS_ACCOUNT_ID}:policy/KarpenterControllerPolicy-${CLUSTER_NAME}" \
  --role-only \
  --approve

export KARPENTER_IAM_ROLE_ARN="arn:aws:iam::${AWS_ACCOUNT_ID}:role/${CLUSTER_NAME}-karpenter"
```

### Create the EC2 Spot Service Linked Role

This step is only necessary if this is the first time you’re using EC2 Spot in this account. More details are available here.

```
aws iam create-service-linked-role --aws-service-name spot.amazonaws.com || true
```

### Install karpenter Helm Chart

```
helm upgrade --install karpenter oci://public.ecr.aws/karpenter/karpenter --version ${KARPENTER_VERSION} --namespace karpenter --create-namespace \
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=${KARPENTER_IAM_ROLE_ARN} \
  --set clusterName=${CLUSTER_NAME} \
  --set clusterEndpoint=${CLUSTER_ENDPOINT} \
  --set aws.defaultInstanceProfile=KarpenterNodeInstanceProfile-${CLUSTER_NAME} \
  --wait
```

### Provisioner

```
kubectl apply -f karpenter/provisioner.yaml
```


## Create Kubeconfig

```
aws eks --region $AWS_DEFAULT_REGION update-kubeconfig --name $CLUSTER_NAME
```

## Delete Karpenter

```
export CLUSTER_NAME="lr4"

helm uninstall karpenter --namespace karpenter
eksctl delete iamserviceaccount --cluster ${CLUSTER_NAME} --name karpenter --namespace karpenter
aws cloudformation delete-stack --stack-name Karpenter-${CLUSTER_NAME}
aws ec2 describe-launch-templates \
    | jq -r ".LaunchTemplates[].LaunchTemplateName" \
    | grep -i Karpenter-${CLUSTER_NAME} \
    | xargs -I{} aws ec2 delete-launch-template --launch-template-name {}

k delete -f ~/src/v2/Setup/nlb-with-tls-termination-1.3.1.yaml
k delete ns karpenter ingress-nginx
eksctl delete ng ${CLUSTER_NAME}-ng --cluster ${CLUSTER_NAME} --drain=false --wait
eksctl delete cluster -f /Users/lrochette/src/lrochette/til/classic/karpenter/cluster.yaml

```
