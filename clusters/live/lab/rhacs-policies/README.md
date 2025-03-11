
# WIP
Based off of [A collection of policy examples for Open Cluster Management.]( https://github.com/open-cluster-management-io/policy-collection/tree/main/policygenerator/policy-sets/community/acs-secure)

1. **Argo CD ApplicationSet**  
   - An ApplicationSet is configured to monitor a Git repository
     
2. **rhacs‑policies Git Repo**  
   - This repository contains all your ACS policy definitions along with a Kustomize configuration.
   - It includes a `kustomization.yaml` that ties together multiple resources.

3. **Kustomization.yaml**  
   - The `kustomization.yaml` file defines how the raw YAML files (manifests) should be assembled.
   - It references a `policyGenerator.yaml` file as one of its generators.

4. **PolicyGenerator.yaml**  
   - This file is an external plugin configuration that takes the raw policy definitions and “wraps” them into ACM‑compatible Policy CRs.
   - It applies defaults (like policy severity, remediation action, and policy sets) and processes individual policy manifests.
  
5. **Final Generated Policy Resources**  
   - The output of the PolicyGenerator (via Kustomize) is a complete set of policies, placements, and bindings that ACM then applies to your clusters.
   - These policies ensure that your Advanced Cluster Security configuration is enforced according to your Git‑declared definitions.

> 💡 **Note:** The **rhacm-hub** namespace is required 
 ```
oc new-project rhacm-hub
```

> 💡 **Note:** Node labels are required
 ```
oc label node <node-name> node-role.kubernetes.io/infra=""
oc label node <node-name> spoke-gitops=true

```

> 💡 **Note:** Enable alpha plugins and the ClusterRoleBinding for **openshift-gitops-policy-admin**
 ```
oc apply -f https://raw.githubusercontent.com/turbra/acm-install-acs/refs/heads/generator/cluster-role.yaml

oc -n openshift-gitops patch argocd openshift-gitops --type merge --patch "$(curl https://raw.githubusercontent.com/turbra/acm-install-acs/refs/heads/generator/argocd-patch.yaml)"
```


