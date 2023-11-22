# How to install the Artifact repository for Argo workflow

this is for a Helm installation of the GitOps Runtime

If you run a workflow and get an error similar to:
```
Error (exit code 1): You need to configure artifact storage. More information on how to do this can be found in the docs: https://argoproj.github.io/argo-workflows/configure-artifact-repository/  
```

## 1- Artifact Repo
add the following block to the resources/RUNTIME/chart/values.yaml

```yaml
gitops-runtime:
  argo-workflows:
    useDefaultArtifactRepo: true  
    useStaticCredentials: false   
    artifactRepository:
      archiveLogs: true
      s3:
        accessKeySecret: {}
        secretKeySecret: {}
        insecure: false
        bucket: csdp-lr            # change as needed
        endpoint: s3.amazonaws.com # change as needed
        region: us-east-1          # change as needed
        roleARN: arn:aws:iam::835357571861:role/lr-s3-wks

```

**Notes:**
  - useDefaultArtifactRepo: not sure why
  - useStaticCredentials: to force the use of SA instead of static credentials along with this block:
  ```yaml
  artifactRepository:
    s3:
      accessKeySecret: {}
      secretKeySecret: {}
  ```

Check the `argo-workflow-controller-configmap` to verify the `artifactRepository` block is present.

Once this done, I had to restart the `argo-workflow-controller` pod to load the new ConfigMap.

Run a new workflow to verify the error message above has disappeared and has ben replace by a permission one like
```
Error (exit code 1): failed to create new S3 client: AccessDenied: User: arn:aws:sts::835357571861:assumed-role/eksctl-onpremlr-nodegroup-runner-NodeInstanceRole-Q8BL8DSP476Z/i-054f107c71b5e1506 is not authorized to perform: sts:AssumeRole on resource: arn:aws:iam::835357571861:role/lr-s3-wks
	status code: 403, request id: 83ab146f-3ff0-4abb-bb4b-65883f4d123f  
```

## AWS Permissions

Those were done using OIDC.

### Check your OIDC

1. Get the cluster OIDC url aka `https://oidc.eks.us-east-1.amazonaws.com/id/A7F0B51A0AA159E93D55D2A438A7052D`

2. get the ARN associated with the OIDC provider for that cluster aka `arn:aws:iam::835357571861:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/A7F0B51A0AA159E93D55D2A438A7052D`

### Create a new Policy

Create a policy named something like `workflow-assume-role-<CLUSTER>` with the following json code:
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "AllowAssume",
			"Effect": "Allow",
			"Action": "sts:AssumeRole",
			"Resource": "*"
		}
	]
}
```

### Create a new Role
this new role of type `custom trust policy` named something like workflow-<CLUSTER> where the trust policy is something like:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "<ARN OF OIDC we checked above>"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringLike": {
                    "oidc.eks.us-east-1.amazonaws.com/id/<ACCOUNT_ID>:sub": "system:serviceaccount:<RUNTIME NAMESPACE>:*"
                }
            }
        }
    ]
}
```

and add the policy we created in the previous step
Finish creating the role and save the ARN aka `arn:aws:iam::835357571861:role/workflow-onpremlr`

### Add trust relationship to role

1. Find the role indicated in the artifactRepository block at the start of this page (`arn:aws:iam::835357571861:role/lr-s3-wks`)
2. edit the trust relationship allow AssumeRole for the one creaed in the previous step (aka `arn:aws:iam::835357571861:policy/workflow-assume-role-onpremlr``) something like

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        },
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::835357571861:role/workflow-onpremlr"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

## Finish the workflow configmap setup

let's go back to our values file to reference those new roles

1. let's define the ServiceAccount to use for running the workflows
```yaml
gitops-runtime:
  argo-workflows:
    controller:
      workflowDefaults:
        spec:
          serviceAccountName: workflows-default
```
2. Add annotations to use the role created
```yaml
gitops-runtime:
  argo-workflows:
    server:
      serviceAccount:
        annotations:
          eks.amazonaws.com/role-arn: "arn:aws:iam::835357571861:role/workflow-onpremlr"
    workflow:
      serviceAccount:
        create: true
        name: workflows-default
        annotations:
          eks.amazonaws.com/role-arn: "arn:aws:iam::835357571861:role/workflow-onpremlr"    
```
4. Commit the code to sync again and update the runtime

The final version of my change for the workflow looks like:
```yaml
gitops-runtime:
  argo-workflows:
    controller:
      workflowDefaults:
        spec:
          serviceAccountName: workflows-default
    useDefaultArtifactRepo: true
    useStaticCredentials: false    
    artifactRepository:
      archiveLogs: true
      s3:
        accessKeySecret: {}
        secretKeySecret: {}
        insecure: false
        bucket: csdp-lr
        endpoint: s3.amazonaws.com #change as needed
        region: us-east-1 #change as needed
        roleARN: arn:aws:iam::835357571861:role/lr-s3-wks
    server:
      serviceAccount:
        annotations:
          eks.amazonaws.com/role-arn: "arn:aws:iam::835357571861:role/workflow-onpremlr"
    workflow:
      serviceAccount:
        create: true
        name: workflows-default
        annotations:
          eks.amazonaws.com/role-arn: "arn:aws:iam::835357571861:role/workflow-onpremlr"    

```
