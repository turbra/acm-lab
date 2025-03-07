
# **Overview: Argo CD, ACM, and ACS**

## 1. Introduction

This guide explains how Red Hat Advanced Cluster Security (ACS) is deployed using Red Hat Advanced Cluster Management (ACM) and Argo CD (GitOps).  
- **Argo CD** continuously monitors a Git repository and applies the Kubernetes resources (in this case, the ACM Subscription and related configurations).  
- **ACM** subscribes to these resources, creating the necessary **Policies** that install and manage ACS across one or more clusters.  
- **ACS** (Advanced Cluster Security) provides container security, compliance checks, and runtime protection once deployed.

## 2. High-Level Architecture

1. **Git Repository**: Stores the YAML manifests needed to install ACS, including operator subscriptions, policies, and placements.  
2. **Argo CD**:  
   - Points to the Git repo containing the ACM Subscription definition.  
   - Applies these Subscription resources on the **hub cluster** (the cluster hosting ACM).  
3. **ACM**:  
   - Uses the Subscription to fetch policies (and their payloads) from Git.  
   - Enforces where those policies apply via **PlacementRules** or **Placements**.  
4. **ACS**:  
   - The actual security platform installed on the selected clusters (hub and/or managed).  
   - Comprises an operator, central services, and sensors.

**Key Namespaces**  
- **Argo CD Namespace** (e.g. `openshift-gitops`): Where Argo CD itself runs.  
- **ACM Namespace** (e.g. `acs-install-policies`): Holds the Subscription, Channel, and Policy CRs.  
- **Target Cluster Namespaces** (e.g. `stackrox`, `openshift-compliance`): Where ACS components end up running on each cluster.

## 3. Deployment Sequence (Simplified)

1. **Argo CD Application Creation**  
   - You define an **Argo CD Application** that points to the Git repository (e.g., `acm-install-acs`).  
   - This Application targets the **hub cluster** namespace where you want to place the Subscription objects (e.g., `acs-install-policies`).  
   - Once synced, Argo CD creates:
     - A **Namespace** (`acs-install-policies`) for the subscription resources.  
     - A **Channel** object, referencing the Git repo URL.  
     - A **Subscription** object, specifying which branch/path to sync and a **PlacementRule** to apply it on the hub.

2. **ACM Subscription Pulls Policies**  
   - ACM sees the new Subscription in the `acs-install-policies` namespace.  
   - The Subscription’s Git channel points to the repository containing ACS policies, operator manifests, etc.  
   - ACM fetches these resources, generates **Policy** CRs for installing ACS (operator, central, sensor, compliance, etc.).

3. **Policies & Placement Deploy ACS**  
   - Each **Policy** defines the Kubernetes objects needed (e.g., operator subscriptions, custom resources).  
   - **Placement** (or PlacementRule) inside the policy or associated with the policy determines which clusters get ACS.  
   - ACM applies these Policy manifests on each targeted cluster (hub or managed).  
   - The ACS Operator and components spin up in the appropriate namespaces (e.g., `stackrox`, `openshift-compliance`).

4. **Continuous Compliance & Monitoring**  
   - ACM tracks policy compliance for each cluster (Compliant/NonCompliant).  
   - If any cluster drifts from the desired state, ACM enforces the policy (reinstalling or correcting missing resources).  
   - ACS itself provides UI, runtime checks, and advanced security monitoring once up and running.

> 💡Note: labels are required
 ```
oc label node<node-name> node-role.kubernetes.io/infra=""
oc label managedcluster local-cluster app=acs-install-policies
```
