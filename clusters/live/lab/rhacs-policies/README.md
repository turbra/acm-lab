
# WIP
Based off of [A collection of policy examples for Open Cluster Management.]( https://github.com/open-cluster-management-io/policy-collection/tree/main/policygenerator/policy-sets/community/acs-secure)

> 💡Note: rhacm-hub namespace is required
 ```
oc new-project rhacm-hub
```

> 💡Note: node labels are required
 ```
oc label node <node-name> node-role.kubernetes.io/infra=""
oc label node <node-name> spoke-gitops=true

```

> 💡Note: enable alpha plugins and CRB for openshift-gitops-policy-admin
 ```
oc apply -f https://raw.githubusercontent.com/turbra/acm-install-acs/refs/heads/generator/cluster-role.yaml

oc -n openshift-gitops patch argocd openshift-gitops --type merge --patch "$(curl https://raw.githubusercontent.com/turbra/acm-install-acs/refs/heads/generator/argocd-patch.yaml)"
```

