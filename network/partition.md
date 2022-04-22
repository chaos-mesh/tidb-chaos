# Test Network Chaos on TiDB

## Network Partition

### Description

Test the availability of the TiDB cluster in network partition scenarios.

### Hypothesis

When a network partition occurs between a TiKV node and other TiKV nodes in the cluster, the QPS/TPS drop significantly and then recover to normal levels, and the data is consistent.

### Preparation

1. Install chaos-mesh.
2. A TiDB cluster on K8s with at least 3 TiKV Pods, deploy with [TiDB operator](https://docs.pingcap.com/tidb-in-kubernetes/stable/tidb-operator-overview).
3. Running payload on TiDB cluster.

### Quick start

1. Chaos Mesh fault YAML configuration: 

```YAML
kind: NetworkChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  namespace: tidb-cluster
  name: network-partition
spec:
  selector:
    pods:
      tidb-cluster:
        - basic-tikv-0
  mode: all
  action: partition
  duration: 10m
  direction: both
  target:
    selector:
      pods:
        tidb-cluster:
          - basic-tikv-1
          - basic-tikv-2
    mode: all
```

The YAML configuration will cut off the network between Pod `basic-tikv-0` and `basic-tikv-1`, `basic-tikv-2`, saving it into file network-partition.yaml.

2. Using Kubectl to create the experiment:

```
kubectl create -f network-partition.yaml
```

3. Verifying TiDB's status:

    Check QPS & TPS of TiDB in Grafana, and check the data in TiDB.
    <!-- TODO: Add some Grafana picture -->

4. Result:

Judge whether the hypothesis is correct or not based on the results of the test process.

### More example

You can test more scenarios by using Chaos Mesh. For example:

- Network partition occurs between TiDB nodes.
- Network partition occurs between PD nodes.
- Network partition occurs between TiDB and TiKV.
- Network partition occurs between PD and TiKV.

All you need to do is adjust the `selector` and `target` in the YAML configuration. For network partition that occurs between TiDB and TiKV, the YAML configuration looks like the below:

```YAML
kind: NetworkChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  namespace: tidb-cluster
  name: network-partition
spec:
  selector:
    namespaces:
      - tidb-cluster
    labelSelectors:
      app.kubernetes.io/component: tidb
  mode: one
  action: partition
  duration: 10m
  direction: both
  target:
    selector:
      namespaces:
        - tidb-cluster
      labelSelectors:
        app.kubernetes.io/component: tikv
    mode: one
```
