# acm-install-acs
Deploy ACS to ACM hub and managed clusters using ACM policygenerator

Notes:
- User needs to be added to open-cluster-management:subscription-admin ClusterRoleBinding
```
oc patch clusterrolebinding open-cluster-management:subscription-admin --type merge -p '{"subjects": [{"kind": "User", "apiGroup": "rbac.authorization.k8s.io", "name": "admin"}]}'
```
- If you plan on using ArgoCD to deploy, you'll need to add the following service account
```
oc patch clusterrolebinding open-cluster-management:subscription-admin --type merge -p '{"subjects": [{"kind": "User", "apiGroup": "rbac.authorization.k8s.io", "name": " system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller"}]}'
```
```
oc get clusterrolebinding open-cluster-management:subscription-admin -o yaml
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: open-cluster-management:subscription-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: open-cluster-management:subscription-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: admin
```

- Add user to system:image-puller role for project

```yaml
    kind: ClusterRoleBinding  
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: sky-image-pull-binding
    subjects:
      - kind: User
        apiGroup: rbac.authorization.k8s.io
        name: admin
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: 'system:image-puller'
```
A subscription to this repo might look like this:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: acs-install-policies
---
apiVersion: app.k8s.io/v1beta1
kind: Application
metadata:
  name: acs-install-policies
  namespace: acs-install-policies
spec:
  componentKinds:
    - group: apps.open-cluster-management.io
      kind: Subscription
  selector:
    matchExpressions:
      - key: app
        operator: In
        values:
          - acs-install-policies
---
apiVersion: apps.open-cluster-management.io/v1
kind: Channel
metadata:
  annotations:
    apps.open-cluster-management.io/reconcile-rate: medium
  name: acs-install-policies-channel
  namespace: acs-install-policies
spec:
  type: Git
  pathname: https://github.com/turbra/acm-install-acs.git
---
apiVersion: apps.open-cluster-management.io/v1
kind: Subscription
metadata:
  annotations:
    apps.open-cluster-management.io/git-branch: main
    # apps.open-cluster-management.io/git-path: N/A
    apps.open-cluster-management.io/reconcile-option: merge
  labels:
    app: acs-install-policies
  name: acs-install-policies-subscription
  namespace: acs-install-policies
spec:
  channel: acs-install-policies/acs-install-policies-channel
  placement:
    placementRef:
      kind: PlacementRule
      name: acs-install-policies-placement
---
apiVersion: apps.open-cluster-management.io/v1
kind: PlacementRule
metadata:
  labels:
    app: acs-install-policies
  name: acs-install-policies-placement
  namespace: acs-install-policies
spec:
  clusterSelector:
    matchLabels:
      name: local-cluster
```

> 💡Note: 
your clusters will require the appropriate labels if using the subscription example included above

```
oc label managedcluster local-cluster app=acs-install-policies
```
