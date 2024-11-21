# **DKG Components Deployment Guide**

This document provides instructions for deploying the Distributed Knowledge Graph (DKG) components in a Kubernetes cluster. The deployment ensures all components are correctly applied in the required order, starting with the namespace and dependencies.

## **Prerequisites**

1. **Kubernetes Cluster**:
   Ensure you have access to a running Kubernetes cluster.

2. **Kubectl**:
   Install and configure the Kubernetes CLI tool, `kubectl`, with access to the cluster.

3. **Docker Images**:
   Ensure all required container images are accessible.

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

## **Deploying components**

### Run the deployment script to run all components.
Use the provided `run_dkg_components.sh` script to deploy all components in the correct order.

### Access the KubeQuery API
Access kubequery at `$CONTROL_PLANE_IPADDR:30094`, where `$CONTROL_PLANE_IPADDR` is the IP address of the control plane of the main cluster. Use the `/swagger/` for documentation.

---

## **Adding Another Cluster**

To integrate another cluster into the Distributed Knowledge Graph (DKG), follow these steps:

1. **Clone the Repository**

2. **Modify the KubeInsights Configuration**:
   - Edit the `kubeinsights.yaml` file in the cloned repository:
     - Update the `ConfigMap` section with the following changes:
       - Set the `KAFKA_BROKER_URL` to `$CONTROL_PLANE_IPADDR:30093`, where `$CONTROL_PLANE_IPADDR` is the IP address of the control plane of the main cluster.
       - Update the `KAFKA_TOPIC` to the name of the new cluster (e.g., `cluster2`).

3. **Deploy KubeInsights**:
   - Apply the `kubeinsights.yaml` file to deploy KubeInsights in the new cluster:
     ```bash
     kubectl apply -f kubeinsights.yaml
     ```
