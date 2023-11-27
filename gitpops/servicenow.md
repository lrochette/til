# Service now integration for GitOps


## Resume

```
mutation resumeWorkflow($namespace: String!, $workflowName: String!) {
  resumeWorkflow(namespace: $namespace, workflowName: $workflowName)
}


{
  "namespace": "csdp",
  "workflowName": "corpsite-ci-vcr5s"
}
```

## Stop
```
mutation stopWorkflow($namespace: String!, $workflowName: String!) {
  stopWorkflow(namespace: $namespace, workflowName: $workflowName)
}
{
  "namespace": "csdp",
  "workflowName": "corpsite-ci-vcr5s"
}
```
