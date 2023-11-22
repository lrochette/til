# On-prem installation

[Full Documentation](https://codefresh.io/docs/docs/administration/codefresh-on-prem/)

# After installation

* Be sure to set the segment as `ENTERPRISE` for the account

# Service to externalize

Current built-in mongodb/postgres charts don't have full enterprise capabilities.
(no stateful set option, no pdb, no metrics exporter, etc)

## Postgres

Externalized for CF SaaS using RDS

## MongoDB

Externalized for CF SaaS using Atlas

## Redis
## RabbitMQ
## Consul

# Debug mode

Enable `debugFeature` and `debugUnlimited` flags.

# Troubleshooting

* How to fixed a failed release without rollback

delete the secret named sh.helm.release.v1.cf.v[XXX]

* No access to the UI
  * check the LB and verify the instances are in service,
  * and add AZ if the nodes are in a different one
* Requires https:// protocol or get Server Error


```
Error: Failed to deploy operator chart: UPGRADE of cf FAILED: cannot patch "cf-charts-manager" with kind Job: Job.batch "cf-charts-manager" is invalid: spec.template: Invalid value: core.PodTemplateSpec{ObjectMeta:v1.ObjectMeta{Name:"cf-charts-manager", GenerateName:"", Namespace:""
```

Solution

* Looks like on your cluster old chart-manager job is left.
* Delete it with kubectl
``` k delete job -n onprem cf-charts-manager```
