# SkyWalking

## Helm install

export SKYWALKING_RELEASE_VERSION=4.4.0  # change the release version according to your need
export SKYWALKING_RELEASE_NAME=skywalking  # change the release name according to your scenario
export SKYWALKING_RELEASE_NAMESPACE=skywalking  # change the namespace to where you want to install SkyWalking
export APM_VERSION=9.4.0

based on https://amjadhussain3751.medium.com/step-by-step-detailed-guide-to-setup-apache-skywalking-on-kubernetes-8369e3d93242

### Get Repo and charts

```
git clone https://github.com/apache/skywalking-kubernetes
cd skywalking-kubernetes/chart

helm repo add elastic https://helm.elastic.co
helm dep up skywalking
```

### Install

```
kubectl create ns ${SKYWALKING_RELEASE_NAMESPACE}
helm install ${SKYWALKING_RELEASE_NAME} skywalking \
  -n "${SKYWALKING_RELEASE_NAMESPACE}" \
  --set oap.image.tag=${APM_VERSION} \
  --set oap.storageType=elasticsearch \
  --set ui.image.tag=${APM_VERSION} \
  --set oap.env.SW_NAMESPACE=skywalking
```

#  --set elasticsearch.persistence.enabled=true \



***Notes:***

* `--set elasticsearch.persistence.enabled=true` will create statefulset ES
* `--set oap.env.SW_NAMESPACE=skywalking` is to create the elasticsearch indexes name with given pattern
