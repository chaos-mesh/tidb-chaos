# Test Network Chaos on TiDB

## Network Delay

### Description

Test the availability of TiDB cluster in network delay scenarios.

### Hypothesis

When the network delay between TiDB and TiKV nodes increases by 50ms, the QPS/TPS of TiDB will drop significantly, but it can still provide services normally.

### Preparation

1. Install chaos-mesh
2. A TiDB cluster on k8s with at least 3 tikv POD, deploy with [TiDB operator](https://docs.pingcap.com/tidb-in-kubernetes/stable/tidb-operator-overview).
3. Runing payload on TiDB cluster

### Quick start

1. Chaos Mesh fault YAML configuration:

```YAML
kind: NetworkChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  namespace: tidb-cluster
  name: network-delay
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
  direction: both
  target:
    selector:
      namespaces:
        - tidb-cluster
      labelSelectors:
        app.kubernetes.io/component: tikv
    mode: all
```

Saving the YAML configuration above into file network-delay.yaml

2. Using Kubectl to create the experiment:

```
kubectl create -f network-delay.yaml
```

3. Verifying TiDB's status:

    Check QPS & TPS of TiDB in Grafana.
    <!-- TODO: Add some grafana picture -->

4. Result:

Judge whether the hypothesis is correct or not based on the results of the test process.
