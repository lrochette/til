# How to extract the list of pipelines associated to a specific RE

you can use the Codefresh CLI to get the pipeline names associated with a particular RE:
```
codefresh get pipelines -o json --limit 1000 | jq '.[] | select(.spec.runtimeEnvironment.name=="cf_onprem_eks/runner") | .metadata.name
```
