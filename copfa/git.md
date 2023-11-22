# Git integration support

See [Internal Confluence page](https://codefresh-io.atlassian.net/wiki/spaces/PRODUCT/pages/2422833560/Git+Integration+Support+in+Platform) for details on the state of the different Git Integrations

## Restrictions

There is also additional items that are needed before you can use BitBucket.

 1. If you already installed the Hosted or Hybrid Runtime we will have to Uninstall all Runtimes before hand.
 2. Support will need to update the Backend Database to allow a different Internal Shared Config Repo specification.
   1. We only allow 1 git provider per account at this time.
 3. Once the first two steps are done, there is actually a Hidden flag that will need to be used to allow BitBucket during installation.
   1. In addition to `--provider bitbucket` you will need `--enable-git-providers bitbucket`


### Bitbucket installation

```
cf2 runtime install csdp4 --repo https://bitbucket.org/lrochette/csdp4 \
  --context lr4 --auth-context csdp4 \
  --demo-resources=false \
  --git-token  ${BITBUCKET_TOKEN} \
  --git-user lrochette \
  --ingress-class nginx \
  --ingress-host https://lr4.support.cf-cd.com \
  --namespace csdp4 \
  --provider bitbucket  \
  --provider-api-url https://api.bitbucket.org \
  --shared-config-repo https://bitbucket.org/lrochette/isc \
  --log-level debug
```

### gitHub SaaS installation

```
export OWNER=lrochette
export NAME=csdp4
export CONTEXT=lr4
export DOMAIN=lr4
k config set-context $CONTEXT

cf2 runtime install ${NAME} \
  --repo https://github.com/${OWNER}/${NAME} \
  --context lr4 --auth-context csdp4 \
  --demo-resources=false \
  --provider github --git-token  ${GIT_TOKEN} \
  --git-user lrochette \
  --ingress-class nginx \
  --ingress-host https://${DOMAIN}.support.cf-cd.com \
  --namespace ${NAME} \
  --provider-api-url https://api.github.com \
  --shared-config-repo https://github.com/lrochette/isc4 \
  --log-level debug
```

cf2 integration git add default --runtime csdp4 --provider github --api-url https://api.github.com
cf2 integration git register default --runtime csdp4 --token <your-token>
