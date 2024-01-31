# Service now integration for GitOps

call goes to the ingress https://lrcsdp.support.cf-cd.com/app-proxy/api/graphql

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

# utilities
[Xplore](https://developer.servicenow.com/connect.do#!/share/contents/9650888_xplore_developer_toolkit?v=4.05&t=PRODUCT_DETAILS)
[SN Utils Chrome extension](https://chromewebstore.google.com/detail/sn-utils-tools-for-servic/jgaodbdddndbaijmcljdbglhpdhnjobg?hl=en)
