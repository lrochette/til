# Ingress issues

## Cannot see logs
the ingress seems to be working (can access /workflows) but the logs are not visible.

=> protocol needs to be tcp
if set change
```
service.beta.kubernetes.io/aws-load-balancer-backend-protocol: http
```
to
```
service.beta.kubernetes.io/aws-load-balancer-backend-protocol: tcp
``` 
