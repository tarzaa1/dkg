# **KubeGraph**

KubeGraph enables the modeling of kubernetes objects such as node, pods, deployment, configmaps, etc. and their interconnections as a propoerty graph using 5 containerized components. In this repo you can fine manifest files for each of the components and a simple shell script that ensures they are correctly applied in the required order.

## **Compnents**

1. **KubeInsights**:
   Retrieves the metadata of Kubernetes objects using the kube-api-server and submits updates to a Kafka broker. It also monitors the cluster for any new events (e.g., a node being added or removed, a pod being added, deleted or updated, etc.)

2. **KubeGrapher**:
   Consumes state updates from the event broker as JSON blobs; transforms and stores these updates into Neo4J using a the Cypher query language.

3. **KubeQuery**:
   Implements a RESTful API (with Flask) which interfaces with the Neo4j to facilitate query operations via various endpoints that implement Cypher queries.

---

## **Deploying components on the main cluster**

1. **Setup Metrics Server**:
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
