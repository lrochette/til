# Plugins

Notes about installing Helmfile plugins in a Hybrid runtime, and then in a Hosted one

Installation based on https://github.com/travisghansen/argo-cd-helmfile

## Hybrid runtime
Installing the plugins is done thru the bootstrap/argo-cd/kustomization.yaml.
Here is the code:

``` yaml
apiVersion: kustomize.config.k8s.io/v1beta1
configMapGenerator:
- behavior: merge
  literals:
  - |
    repository.credentials=- passwordSecret:
        key: git_token
        name: autopilot-secret
      url: https://github.com/
      usernameSecret:
        key: git_username
        name: autopilot-secret
  - |
    configManagementPlugins=- name: helmfile
      init:
        command: [sh, -c]
        args: ["/usr/local/bin/argo-cd-helmfile.sh init"]
      generate:
        command: [sh, -c]
        args: ["/usr/local/bin/argo-cd-helmfile.sh generate"]
  name: argocd-cm
kind: Kustomization
namespace: csdp
patchesStrategicMerge:
- |-
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: argocd-repo-server
  spec:
    template:
      spec:
        containers:
        - name: argocd-repo-server
          volumeMounts:
          - mountPath: /usr/local/bin/argo-cd-helmfile.sh
            name: custom-tools
            subPath: argo-cd-helmfile.sh
          - mountPath: /usr/local/bin/helmfile
            name: custom-tools
            subPath: helmfile
          - mountPath: /home/argocd/.local/share/helm
            name: helm-data-home
        volumes:
        - name: custom-tools
          emptyDir: {}
        - name: helm-data-home
          emptyDir: {}
        initContainers:
        - name: download-tools
          image: alpine:3.8
          command: [sh, -c]
          args:
            - wget -qO /custom-tools/argo-cd-helmfile.sh https://raw.githubusercontent.com/travisghansen/argo-cd-helmfile/master/src/argo-cd-helmfile.sh &&
              chmod +x /custom-tools/argo-cd-helmfile.sh &&
              wget -qO /custom-tools/helmfile https://github.com/roboll/helmfile/releases/download/v0.138.7/helmfile_linux_amd64 &&
              chmod +x /custom-tools/helmfile
          volumeMounts:
            - mountPath: /custom-tools
              name: custom-tools
            - mountPath: /helm/data
              name: helm-data-home
resources:
- github.com/codefresh-io/csdp-official/csdp/hybrid/basic/apps/argo-cd?ref=0.1.16

```

Next part is the application itself

``` yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: helmfile-hosted
  namespace: csdp
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://gitlab.com/voting-application/config.git
    targetRevision: master
    path: helm
    plugin:
      name: helmfile
      env:
      - name: HELM_DATA_HOME
        value: /home/argocd/.local/share/helm
  destination:
    name: lr2
    namespace: dev
  syncPolicy:
    automated: {}
    syncOptions:
      - CreateNamespace=true
```

## Hosted runtime

Hosted runtime defition are in https://github.com/codefresh-io/csdp-managed-runtimes/tree/main/runtimes.
File to modify is in <ACCOUNT-xxxxxx>/bootstrap/kustomization.yaml

## Plugin for Helm-based runtime installer

```
# https://github.com/codefresh-io/gitops-runtime-helm/blob/main/charts/gitops-runtime/values.yaml

global:
  codefresh:
    accountId: "6391ff086e38212f874d43f3"

  runtime:
    name: "cf-pf9-prod-k8s"

argo-cd:
  # https://github.com/argoproj/argo-helm/blob/main/charts/argo-cd/values.yaml
  configs:
    cm:
      kustomize.buildOptions: "--enable-helm --load-restrictor=LoadRestrictionsNone"

  extraObjects:
  - apiVersion: v1
    kind: ConfigMap
    metadata:
      name: avp-plugin-config
    data:
      plugin.yaml: |
        apiVersion: argoproj.io/v1alpha1
        kind: ConfigManagementPlugin
        metadata:
          name: argocd-vault-plugin
        spec:
          allowConcurrency: true
          discover:
            find:
              command:
                - find
                - "."
                - -name
                - kustomization.yaml
          generate:
            command:
              - sh
              - "-c"
              - "kustomize build --enable-helm --load-restrictor=LoadRestrictionsNone . | argocd-vault-plugin generate -"

  repoServer:
    volumes:
    - name: avp-plugin-config
      configMap:
        name: avp-plugin-config
    - name: custom-tools
      emptyDir: {}
    - name: cmp-tmp
      emptyDir: {}
    initContainers:
      - name: get-avp-plugin
        image: alpine:3.18
        command: [sh, -c]
        env:
        - name: AVP_VERSION
          value: 1.14.0
        - name: KUSTOMIZE_VERSION
          value: 5.0.1
        - name: HELM_VERSION
          value: 3.11.3
        args:
        - |
          wget -qO kustomize.tar.gz https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv${KUSTOMIZE_VERSION}/kustomize_v${KUSTOMIZE_VERSION}_linux_amd64.tar.gz
          tar xzf kustomize.tar.gz
          chmod +x kustomize
          mv kustomize /custom-tools/
          wget -qO helm.tar.gz https://get.helm.sh/helm-v${HELM_VERSION}-linux-amd64.tar.gz
          tar xzf helm.tar.gz
          chmod +x linux-amd64/helm
          mv linux-amd64/helm /custom-tools/
          wget -qO argocd-vault-plugin https://github.com/argoproj-labs/argocd-vault-plugin/releases/download/v${AVP_VERSION}/argocd-vault-plugin_${AVP_VERSION}_linux_amd64
          chmod +x argocd-vault-plugin
          mv argocd-vault-plugin /custom-tools/
        volumeMounts:
        - mountPath: /custom-tools
          name: custom-tools
    extraContainers:
    - name: avp-plugin
      image: alpine:3.18
      command: [/var/run/argocd/argocd-cmp-server]
      env:
      - name: AVP_TYPE
        value: awssecretsmanager
      - name: AWS_REGION
        value: us-east-1
      securityContext:
        runAsNonRoot: true
        runAsUser: 999
      volumeMounts:
      - mountPath: /var/run/argocd
        name: var-files
      - mountPath: /home/argocd/cmp-server/plugins
        name: plugins
      - mountPath: /home/argocd/cmp-server/config/plugin.yaml
        subPath: plugin.yaml
        name: avp-plugin-config
      - mountPath: /tmp
        name: cmp-tmp
      - mountPath: /usr/local/bin/argocd-vault-plugin
        name: custom-tools
        subPath: argocd-vault-plugin
      - mountPath: /usr/local/bin/kustomize
        name: custom-tools
        subPath: kustomize
      - mountPath: /usr/local/bin/helm
        name: custom-tools
        subPath: helm
```
