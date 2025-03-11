The ACS PolicySet for Secured Clusters contains two PolicySets that will be deployed. The PolicySets install RHACS Secured Clusters onto all OpenShift clusters that are managed by RHACM except for the RHACM hub cluster. The RHACS Secured Cluster component is deployed to the RHACM hub via an ArgoCD applicationset which will pickup the `rhacs-operator` and `rhacs-central` directories.

Based off of [A collection of policy examples for Open Cluster Management.]( https://github.com/open-cluster-management-io/policy-collection/tree/main/policygenerator/policy-sets/community/acs-secure)

---

Prior to applying the PolicySet, perform these steps:
1. OpenShift GitOps must be configured to provide the `--enable-alpha-plugins` flag when you run Kustomize. Run the following command to configure OpenShift GitOps:
 ```
oc -n openshift-gitops patch argocd openshift-gitops --type merge --patch "$(curl https://raw.githubusercontent.com/turbra/acm-install-acs/refs/heads/generator/argocd-patch.yaml)"
```
2. Policies are installed to the rhacm-hub namespace. ArgoCD will apply `mcsbinding.yaml` accordingly.   

---

**Argo CD ApplicationSet**  
   - An ApplicationSet is configured to monitor a Git repository
     
**rhacs‑policies Git Repo**  
   - This repository contains all your ACS policy definitions along with a Kustomize configuration.
   - It includes a `kustomization.yaml` that ties together multiple resources.

**Kustomization.yaml**  
   - The `kustomization.yaml` file defines how the raw YAML files (manifests) should be assembled.
   - It references a `policyGenerator.yaml` file as one of its generators.

**PolicyGenerator.yaml**  
   - This file is an external plugin configuration that takes the raw policy definitions and “wraps” them into ACM‑compatible Policy CRs.
   - It applies defaults (like policy severity, remediation action, and policy sets) and processes individual policy manifests.
  
**Final Generated Policy Resources**  
   - The output of the PolicyGenerator (via Kustomize) is a complete set of policies, placements, and bindings that ACM then applies to your clusters.
   - These policies ensure that your Advanced Cluster Security configuration is enforced according to your Git‑declared definitions.

---

> 💡 The **rhacm-hub** namespace is required 
 ```
oc new-project rhacm-hub
```

> 💡 **Node labels** are required
 ```
oc label node <node-name> node-role.kubernetes.io/infra=""

oc label node <node-name> spoke-gitops=true
```



