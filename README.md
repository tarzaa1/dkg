# **DKG Components Deployment Guide**

This document provides instructions for deploying the Distributed Knowledge Graph (DKG) components in a Kubernetes cluster. The deployment ensures all components are correctly applied in the required order, starting with the namespace and dependencies.

## **Prerequisites**

1. **Kubernetes Cluster**:
   Ensure you have access to a running Kubernetes cluster.

2. **Kubectl**:
   Install and configure the Kubernetes CLI tool, `kubectl`, with access to the cluster.

3. **Docker Images**:
   Ensure all required container images are accessible in your container registry.

4. **Files**:
   The following YAML manifest files should be available in the current directory:
   - `namespace.yaml`: Defines the `dkg` namespace.
   - `neo4j.yaml`: Defines the Neo4j authentication secret, statfulset, and service.
   - `kafka.yaml`: Defines the Zookeeper and Kafka statefulsets and services.
   - `kubeinsights.yaml`: Defines the KubeInsights serviceaccount, clusterrole, rolebinding, deployment, and configmap.
   - `kubegrapher.yaml`: Defines the KubeGrapher deployment and configmap.
   - `kubequery.yaml`: Defines the KubeQuery deployment and nodeport service.
   - `dependencies.yaml`: Defines a serviceaccount, role, and rolebinding used by various components.

---

## **Deploying All Components**

### Run the Deployment Script to run all components.
Use the provided `run_dkg_components.sh` script to deploy all components in the correct order.

## **Adding another Cluster**
