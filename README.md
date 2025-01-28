# **DKG Components Deployment Guide**

This document provides instructions for deploying the Distributed Knowledge Graph (DKG) components in a Kubernetes cluster. The deployment ensures all components are correctly applied in the required order, starting with the namespace and dependencies.

## **Prerequisites**

1. **Kubernetes Cluster**:
   Ensure you have access one or more Kubernetes clusters.

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

## **Deploying components on the main cluster**

1. **Clone the Repository**

2. **Setup Metrics Server**:
    ```sh
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    ```

2. **Run the deployment script**:
   Use the provided `run_dkg_components.sh` script to deploy all components in the correct order.

3. Access the KubeQuery API
   Access kubequery at `$CONTROL_PLANE_IPADDR:30094`, where `$CONTROL_PLANE_IPADDR` is the IP address of the control plane of the main cluster. Use the `/swagger/` endpoint for documentation.

---

## **Adding Another Cluster**

To integrate another cluster into the Distributed Knowledge Graph (DKG), follow these steps:

1. **Expose Kafka**:
   On the current cluster, update the second advertised listener in `kafka.yaml` (line 63) to the ip address of your control plane node.

2. **Login to the control plane of the new cluster**:
   On the new cluster, you will only need to run kubeinsights inside the dkg namespace. You can either clone the entire repo or just copy the files `namespace.yaml` and `kubeinsights.yaml`.

3. **Modify the KubeInsights Configuration**:
   Edit the `kubeinsights.yaml` file:
     - Update the `ConfigMap` section with the following changes:
       - Set the `KAFKA_BROKER_URL` to `$CONTROL_PLANE_IPADDR:30093` (line 86), where `$CONTROL_PLANE_IPADDR` is the IP address of the control plane of the main cluster where kafka is running.
       - Update the `KAFKA_TOPIC` (line 87) to the name of the new cluster (e.g., `cluster2`).

4. **Setup Metrics Server**:
    ```sh
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    ```

5. **Deploy KubeInsights**:
   - Apply the `namespace.yaml` and `kubeinsights.yaml` files to deploy KubeInsights in the new cluster:
     ```bash
     kubectl apply -f namespace.yaml
     kubectl apply -f kubeinsights.yaml
     ```

6. **You can now access the updated graph via the KubeQuery API**
