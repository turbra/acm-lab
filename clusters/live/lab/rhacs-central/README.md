# Advanced Cluster Security GitOps Deployment with ArgoCD

This repository provides a comprehensive GitOps approach using ArgoCD to deploy the Advanced Cluster Security (ACS) Operator, configure ACS Central, and set up a local Secured Cluster. By leveraging ArgoCD's sync-waves, we ensure that resources are applied in a specific order, allowing for seamless deployment and configuration of ACS components.

## Table of Contents

- [Understanding Sync-Waves](#understanding-sync-waves)
- [Deployment Flow](#deployment-flow)
  - [Sync-Wave 0: Preparatory Resources](#sync-wave-0-preparatory-resources)
  - [Sync-Wave 1: Deploy ACS Central](#sync-wave-1-deploy-acs-central)
  - [Sync-Wave 2: Create Init Bundle](#sync-wave-2-create-init-bundle)
  - [Sync-Wave 3: Set Up Secured Cluster](#sync-wave-3-set-up-secured-cluster)

## Understanding Sync-Waves

[ArgoCD's sync-waves](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/) feature allows us to control the order in which Kubernetes manifests are applied. Resources are synchronized in ascending order based on the `argocd.argoproj.io/sync-wave` annotation. Resources with the same sync-wave are applied in parallel, while resources with higher sync-waves wait for lower ones to complete.

## Deployment Flow

### Sync-Wave 0: Preparatory Resources

**Purpose**: Set up namespaces and foundational components required for the ACS Operator and ACS Central.

**Manifests**:

- **ACS Operator Namespace** (`rhacs-operator/namespace.yaml`)
- **OperatorGroup** (`rhacs-operator/operatorgroup.yaml`)
- **Subscription** (`rhacs-operator/subscription.yaml`)
- **ACS Instance Namespace** (`rhacs-central/namespace.yaml`)

**Annotation**:

~~~
argocd.argoproj.io/sync-wave: "0"
~~~

### Sync-Wave 1: Deploy ACS Central

**Purpose**: Deploy the ACS Central instance after the operator is installed and namespaces are created.

**Manifests**:

- **ACS Central** (`rhacs-central/central.yaml`)
- **ServiceAccount for Init Bundle Job** (`rhacs-central/create-cluster-init-bundle-sa.yaml`)

**Annotation**:

~~~
argocd.argoproj.io/sync-wave: "1"
~~~

### Sync-Wave 2: Create Init Bundle

**Purpose**: Generate an init bundle required for registering the Secured Cluster with ACS Central.

**Manifests**:

- **Init Bundle Job** (`rhacs-central/create-cluster-init-bundle-job.yaml`)

**Annotation**:

~~~
argocd.argoproj.io/sync-wave: "2"
~~~

### Sync-Wave 3: Set Up Secured Cluster

**Purpose**: Deploy the Secured Cluster components that connect to ACS Central using the generated init bundle.

**Manifests**:

- **Secured Cluster** (`rhacs-central/secured-cluster.yaml`)

**Annotation**:

~~~
argocd.argoproj.io/sync-wave: "3"
~~~
