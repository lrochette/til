csdp2
brew install argocd-autopilot
export GIT_REPO=https://github.com/lrochette/argocd

argocd-autopilot repo bootstrap

password: WkxeSulGJ-XOzCoL

# k apply -f ~/src/v2/nadog/argocd-ingress.yaml
kubectl port-forward -n argocd svc/argocd-server 8080:80 &

Argo: https://localhost:8080/

# Create Project and app
argocd-autopilot project create default
argocd-autopilot app create globex \
  --app https://github.com/lrochette/CorpSite_gitops.git \
  --project default \
  --dest-namespace demo \
  --type dir

#
# Manual creation
#
name: globex
project: default
sync: manual
Repo: https://github.com/lrochette/CorpSite_gitops.git
path: ./
CLuster: URL in-cluster
namespace: demo

Click sync
kubectl port-forward -n demo svc/globex 8100:80 &

Change image
wait for out of sync
SYnc again

Voila

Argo: https://localhost:8080/
Globex: https://localhost:8100/


1.0.0 image   => All FR images
1.1.0 image => ALL AZ

Clean:
argocd-autopilot app delete globex --project default

argocd-autopilot repo uninstall
k delete app -n argocd globex
k delete deployment -n demo globex
k delete service -n demo globex
k delete ing -n demo globex
