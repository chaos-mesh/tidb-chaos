# Test Network Chaos on TiDB

## Network Reorder

### Description

Test the availability of the TiDB cluster in network package reorder scenarios.

### Hypothesis

When the network package reorder occurs between TiDB and TiKV nodes, the QPS/TPS of TiDB will drop significantly, but it can still provide services normally.

### Preparation

1. Install chaos-mesh.
2. A TiDB cluster on k8s with at least 3 TiKV Pods, deploy with [TiDB operator](https://docs.pingcap.com/tidb-in-kubernetes/stable/tidb-operator-overview).
3. Running payload on TiDB cluster.

### Quick start

1. Chaos Mesh fault YAML configuration:

```YAML
kind: NetworkChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  namespace: tidb-cluster
  name: network-reorder
spec:
  selector:
    namespaces:
      - tidb-cluster
    labelSelectors:
      app.kubernetes.io/component: tidb
  mode: all
  action: delay
  duration: 10m
  delay:
    latency: 50ms
    correlation: '0'
    jitter: 0ms
    reorder:
      reorder: 50
      correlation: 50
      gap: 5
  direction: both
  target:
    selector:
      namespaces:
        - tidb-cluster
      labelSelectors:
        app.kubernetes.io/component: tikv
    mode: all
```

Saving the YAML configuration above into file network-reorder.yaml.

2. Using Kubectl to create the experiment:

```
kubectl create -f network-reorder.yaml
```

3. Verifying TiDB's status:

    Check QPS & TPS of TiDB in Grafana.
    <!-- TODO: Add some Grafana picture -->

4. Result:

Judge whether the hypothesis is correct or not based on the results of the test process.

### More example

You can test more scenarios by using Chaos Mesh. For example:

- Network package reorder occurs between TiDB and TiKV.
- Network package reorder occurs between TiKV and PD.
- Network package reorder occurs between TiKV nodes.
- Network package reorder occurs between PD nodes.

All you need to do is adjust the `selector` and `target` in the YAML configuration. For inject network package reorder between TiKV nodes, the YAML configuration looks like the below:

```YAML
kind: NetworkChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  namespace: tidb-cluster
  name: network-reorder
spec:
  selector:
    pods:
      tidb-cluster:
        - basic-tikv-0
  mode: all
  action: delay
  duration: 10m
  delay:
    latency: 50ms
    correlation: '0'
    jitter: 0ms
    reorder:
      reorder: 50
      correlation: 50
      gap: 5
  direction: both
  target:
    selector:
      pods:
        tidb-cluster:
          - basic-tikv-1
          - basic-tikv-2
    mode: all
```
