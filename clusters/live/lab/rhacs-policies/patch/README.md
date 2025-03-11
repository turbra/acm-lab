In order for OpenShift GitOps to have access to the policy generator when you run Kustomize, an Init Container is required to copy the policy generator binary from the RHACM Application Subscription container image to the OpenShift GitOps container that runs Kustomize. Additionally, OpenShift GitOps must be configured to provide the --enable-alpha-plugins flag when you run Kustomize.

 ```
oc -n openshift-gitops patch argocd openshift-gitops --type merge --patch "$(curl https://raw.githubusercontent.com/turbra/acm/refs/heads/main/clusters/live/lab/rhacs-policies/patch/argocd-patch.yaml)"
```
