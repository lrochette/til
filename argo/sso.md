# SSO setup for Argo CD

How to integrate ArgoCD with Active Directory and assign roles and permissions such as "read-only", visibility to only certain projects or applications, ...

## ArgoCD with LDAP as an exmaple

Since Argo CD uses Dex, the [Dex documentation page](https://dexidp.io/docs/connectors/ldap/#example-searching-a-active-directory-server-with-groups) is probably the most useful one for this. But it is high level.

1. Edit the `argocd-cm` configmap and set the following...

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  url: https://argocd.example.com # This is the URL to the Argo CD ingress
  dex.config: |
    connectors:
    - type: ldap
      id: ldap
      name: LDAP
      config:
        host: ldap.example.com:636
        bindDN: "$dex.ldap.bindDN" #This references a secret
        bindPW: "$dex.ldap.bindPW" #This references a secret
        rootCAData: LS0tLS... # A base64 encoded rootCA cert for the server. you can also store this in a secret
        userSearch:
          baseDN: cn=users,cn=accounts,dc=example,dc=com
          filter: "(objectClass=posixAccount)"
          username: uid
          idAttr: uid
          emailAttr: mail
          preferredUsernameAttr: uid
        groupSearch:
          baseDN: cn=groups,cn=accounts,dc=example,dc=com
          filter: "(|(objectClass=posixGroup)(objectClass=group))"
          userAttr: DN
          groupAttr: member
          nameAttr: cn
```

2. Update `argocd-secret` secret to have the values of the above referenced settings

```
apiVersion: v1
kind: Secret
metadata:
  name: argocd-secret
  namespace: argocd
type: Opaque
data:
  dex.ldap.bindDN: dWlkP... # Base64 encoding of something like: uid=robotreadonly,cn=users,cn=accounts,dc=example,dc=com
  dex.ldap.bindPW: V2hvb... # Base64 encoding of the password of that user
```

3. Update your `argocd-rbac-cm`  configmap with your group mappings. By default Argo CD only has two roles `admin` and `readonly`  you have to define others (you can get VERY granular). The following config sets anyone that is in the "admin" group in Active Directory gets "role:admin" on Argo CD. Everyone else gets Read Only access. I recommend reading [the RBAC doc](https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/) if you want to setup anything beyond the built-in roles (you can also scope RBAC at the `AppProject` level)  

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.csv: |
    g, admin, role:admin
  policy.default: role:readonly
```

## Notes
Things to look out for: Argo CD won't work well at the user level, it likes working with groups. So whatever bind user you create on Active Directory needs to be able to pull down group membership for the users logging in.
