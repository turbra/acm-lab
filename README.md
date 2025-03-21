This repository provides a GitOps-based deployment for Red Hat Advanced Cluster Security (RHACS) using Argo CD. It can be used for both testing in a lab environment and for existing ACM deployments with managed clusters. The repository deploys several critical components to the hub cluster and uses ACM Policy Sets to distribute security policies to all managed OpenShift clusters.

## Assumptions:
- Infra node labels for RHACS-Central
  - `oc label node <node_name> node-role.kubernetes.io/infra=""`
- OpenShift GitOps Operator has been installed
- ArgoCD ServiceAccount `openshift-gitops-argocd-application-controller` has `ClusterRole` of `cluster-admin`
  - `oc adm policy add-cluster-role-to-user cluster-admin -z openshift-gitops-argocd-application-controller -n openshift-gitops`
- Namespace `rhacm-hub` exists
  - `oc new-project rhacm-hub`
- Managed Cluster(s) have a label of `spoke-gitops=true`

---

## Key Components Deployed via Argo CD ApplicationSet

The Argo CD ApplicationSet in this repository generates Applications that deploy the following resources on the **hub cluster**:

- **rhacs-operator**:  
  The ACS operator is deployed in the hub cluster to manage and coordinate the security components.

- **rhacs-central**:  
  This component installs the central services (for example, the central console, API, and related resources) that are part of the ACS solution.

- **rhacs-secured-cluster**:  
  This resource is used to register and secure the hub cluster as a managed cluster, enabling centralized compliance and security policy enforcement.

- **ACM Policy Sets** (2x):  
  1. **acs-sensors-hub-info** – Configures Advanced Cluster Security components that are used to set up the hub for securing each managed cluster.  
  2. **acs-sensor-clusters** – Distributes ACS components to all OpenShift managed clusters to ensure cluster security.

## How It Works

1. **Argo CD ApplicationSet:**  
   - An ApplicationSet monitors the Git repository and generates individual Argo CD Applications based on the defined folder structure.  
   - This ensures that each component is deployed in the correct namespace on the hub cluster.

2. **GitOps Pipeline:**  
   - The repository uses a combination of Kustomize and the PolicyGenerator plugin to assemble and generate the final manifests.

3. **Policy Generation and Distribution:**  
   - A **PolicyGenerator** configuration processes raw policy definitions (manifests) and wraps them into ACM‑compatible Policy Custom Resources.  
   - Two distinct Policy Sets are defined:
     - **acs-sensors-hub-info:** Targets the hub cluster, ensuring that necessary ACS components are installed and configured to secure managed clusters.
     - **acs-sensor-clusters:** Targets all OpenShift managed clusters (using label selectors) to deploy distributed security policies.

4. **ACM and Managed Clusters:**  
   - Once the policies are generated and applied to the hub cluster, ACM uses them to enforce security and compliance on all connected managed clusters.
   - Managed clusters are selected based on labels (for example, `spoke-gitops=true`) so that policies are distributed accordingly.

## Use Cases

- **Lab Testing:**  
  Quickly deploy and test ACS components in a lab environment using GitOps principles. The repository simplifies re-deployment and iterative testing through Argo CD.

- **Production ACM Deployment:**  
  In an existing ACM setup, use this repository to deploy and manage ACS components on the hub cluster and propagate security policies to all managed clusters automatically.

## Summary

- **Deployment via Argo CD:** An ApplicationSet is used to deploy the RHACS operator, central services, and secured cluster configuration to the hub cluster.
- **Policy Sets:** Two ACM Policy Sets are generated to:
  - Set up the hub for securing managed clusters.
  - Distribute ACS security policies to all OpenShift managed clusters.
- **GitOps Driven:** Changes to policy definitions and configurations in Git are automatically propagated to the clusters, ensuring consistency and compliance.

This setup provides a robust, GitOps-based workflow for deploying and managing Advanced Cluster Security with ACM, whether you’re running a lab environment or a full production deployment.

---

# Example: Argo CD ApplicationSet

Below is an example ApplicationSet that generates Applications from the repository. This ApplicationSet scans all subdirectories under clusters/live/lab/ (excluding others that are not needed), and deploys them into namespaces named after their folder:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: cluster-configs
  namespace: openshift-gitops
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
  - git:
      repoURL: https://github.com/turbra/acm.git
      revision: main
      directories:
      - path: clusters/live/lab/*
  template:
    metadata:
      name: '{{.path.basename}}'
      finalizers:
        - resources-finalizer.argocd.argoproj.io
    spec:
      project: default
      source:
        repoURL: https://github.com/turbra/acm.git
        targetRevision: main
        path: '{{.path.path}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{.path.basename}}'
      ignoreDifferences:
        - group: admissionregistration.k8s.io
          kind: MutatingWebhookConfiguration
          jsonPointers:
            - /webhooks/0/clientConfig/caBundle
      syncPolicy:
        automated:
          prune: true
          allowEmpty: true
```
