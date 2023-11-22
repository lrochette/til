# Codefresh GitOps Runtime

## Install a runtime with CLI

```
export OWNER=lrochette
export NAME=csdp4
export CONTEXT=lr4
export DOMAIN=lr4
k config set-context $CONTEXT

# create ingress
k apply -f ~/src/v2/Setup/nlb-with-tls-termination-1.3.1.yaml
k annotate ingressclass nginx ingressclass.kubernetes.io/is-default-class=true
k get svc -n ingress-nginx ingress-nginx-controller

cf2 runtime install ${NAME} \
  --repo https://github.com/${OWNER}/${NAME} \
  --provider github --git-token $GIT_TOKEN \
  --context ${CONTEXT} \
  --demo-resources=false \
  --ingress-class=nginx \
  --ingress-host="https://${DOMAIN}.support.cf-cd.com" \
   --insecure-ingress-host \
   --skip-cluster-checks

```
