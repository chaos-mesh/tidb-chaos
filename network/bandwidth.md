# Test Network Chaos on TiDB

## Network Bandwidth

### Description

Test the availability of the TiDB cluster in network bandwidth scenarios.

### Hypothesis

When limiting the network bandwidth between TiDB and TiKV to 1mbps, the QPS/TPS of TiDB will drop significantly, but it can still provide services normally.

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
  name: network-bandwidth
spec:
  selector:
    namespaces:
      - tidb-cluster
    labelSelectors:
      app.kubernetes.io/component: tidb
  mode: all
  action: bandwidth
  duration: 10m
  bandwidth:
    rate: 1mbps
    limit: 10000
    buffer: 10000
  direction: both
  target:
    selector:
      namespaces:
        - tidb-cluster
      labelSelectors:
        app.kubernetes.io/component: tikv
    mode: all
```

Saving the YAML configuration above into file network-bandwidth.yaml.

2. Using Kubectl to create the experiment:

```
kubectl create -f network-bandwidth.yaml
```

3. Verifying TiDB's status:

    Check QPS & TPS of TiDB in Grafana.
    <!-- TODO: Add some Grafana picture -->

4. Result:

Judge whether the hypothesis is correct or not based on the results of the test process.

### More example

You can test more scenarios by using Chaos Mesh. For example:

- Limiting the network bandwidth between TiDB and PD.
- Limiting the network bandwidth between PD and TiKV.
- Limiting the network bandwidth between TiDB nodes.
- Limiting the network bandwidth between TiKV nodes.

All you need to do is adjust the `selector` and `target` in the YAML configuration. For limiting the network bandwidth between TiKV nodes, the YAML configuration looks like the below:

```YAML
kind: NetworkChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  namespace: tidb-cluster
  name: network-bandwidth
spec:
  selector:
    pods:
      tidb-cluster:
        - basic-tikv-0
  mode: all
  action: bandwidth
  duration: 10m
  bandwidth:
    rate: 1mbps
    limit: 10000
    buffer: 10000
  direction: both
  target:
    selector:
      pods:
        tidb-cluster:
          - basic-tikv-1
          - basic-tikv-2
    mode: all
```
