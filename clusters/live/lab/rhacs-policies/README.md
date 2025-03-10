
# WIP

> 💡Note: labels are required
 ```
oc label node<node-name> node-role.kubernetes.io/infra=""
oc label managedcluster local-cluster app=acs-install-policies
```

> 💡Note: enable alpha plugins and CRB for openshift-gitops-policy-admin
 ```
oc apply -f https://raw.githubusercontent.com/turbra/acm-install-acs/refs/heads/generator/cluster-role.yaml

oc -n openshift-gitops patch argocd openshift-gitops --type merge --patch "$(curl https://raw.githubusercontent.com/turbra/acm-install-acs/refs/heads/generator/argocd-patch.yaml)"
```
