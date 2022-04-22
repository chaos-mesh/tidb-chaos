# Test Network Chaos on TiDB

## Network Loss

### Description

Test the availability of the TiDB cluster in network package loss scenarios.

### Hypothesis

When the network package loss occurs between TiKV nodes, the QPS/TPS of TiDB will drop significantly, but it can still provide services normally.

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
  namespace: chaos-testing
  name: network-loss
spec:
  selector:
    pods:
      tidb-cluster:
        - basic-tikv-0
  mode: all
  action: loss
  loss:
    loss: '10'
    correlation: '50'
  direction: to
  target:
    selector:
      pods:
        tidb-cluster:
          - basic-tikv-1
          - basic-tikv-2
    mode: all
```

Saving the YAML configuration above into file network-loss.yaml.

2. Using Kubectl to create the experiment:

```
kubectl create -f network-loss.yaml
```

3. Verifying TiDB's status:

    Check QPS & TPS of TiDB in Grafana.
    <!-- TODO: Add some Grafana picture -->

4. Result:

Judge whether the hypothesis is correct or not based on the results of the test process.

### More example

You can test more scenarios by using Chaos Mesh. For example:

- Network package loss occurs between TiDB and TiKV.
- Network package loss occurs between TiKV and PD.
- Network package loss occurs between TiDB nodes.
- Network package loss occurs between PD nodes.

All you need to do is adjust the `selector` and `target` in the YAML configuration. For injecting network package loss between TiDB and TiKV, the YAML configuration looks like below:

```YAML
kind: NetworkChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  namespace: chaos-testing
  name: network-loss
spec:
  selector:
    namespaces:
      - tidb-cluster
    labelSelectors:
      app.kubernetes.io/component: tidb
  mode: all
  action: loss
  loss:
    loss: '10'
    correlation: '50'
  direction: to
  target:
    selector:
      namespaces:
        - tidb-cluster
      labelSelectors:
        app.kubernetes.io/component: tikv
    mode: all
```
