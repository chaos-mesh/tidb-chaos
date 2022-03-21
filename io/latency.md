# Test IO Chaos on TiDB
## IO Latency

### Description
Test the availability of TiDB cluster in IO Chaos with IO latency scenarios.

### Hypothesis

When about one third of TiKV pod in TiDB cluster suffered a high IO latency, the TiDB cluster can still provide services normally.

### Preparation

1. Install chaos-mesh
2. A TiDB cluster on k8s with at least 3 tikv POD, deploy with [TiDB operator](https://docs.pingcap.com/tidb-in-kubernetes/stable/tidb-operator-overview).
3. Runing payload on TiDB cluster

### Quick start
1. Chaos Mesh fault YAML configuration:
```YAML
kind: IOChaos
apiVersion: chaos-mesh.org/v1alpha1
metadata:
  namespace: ${TiDB-Namespace}
  name: IO-Latency
spec:
  selector:
    namespaces:
      - ${TiDB-Namespace}
    pods:
       ${TiDB-Namespace}:
        - Some-Of-TiKV-PODS
  mode: all
  action: latency
  delay: 100ms
  methods:
    - read
    - write
  volumePath: /var/lib/tikv
  containerName: tikv
```
2. Using Kubectl to create the experiment:
```
kubectl create -f IOChaos-Latency.yaml
```
3. Verifying TiDB's status:

    Check QPS & TPS of TiDB in Grafana.
    <!-- TODO: Add some grafana picture -->
4. Result:

Judge whether the hypothesis is correct or not based on the results of the test process.

### More example

In practise, a continues IO latency chaos with some intervals is a nice example.


Continues IO Latency intervals:
```YAML
apiVersion: chaos-mesh.org/v1alpha1
kind: Workflow
metadata: {}
spec:
  entry: entry
  templates:
    - name: entry
      templateType: Serial
      children:
        - IO-Latency-1
        - Interval-1
        - IO-Latency-1
        - Interval-1
        - IO-Latency-1
        - Interval-1
    - name: IO-Latency-1
      templateType: IOChaos
      deadline: 5m
      ioChaos:
        selector:
          namespaces:
            - ${TiDB-Namespace}
          pods:
            ${TiDB-Namespace}:
              - Some-Of-TiKV-PODS
        mode: all
        action: latency
        delay: 100ms
        volumePath: /var/lib/tikv
        containerName: tikv
        methods:
          - read
          - write
    - name: Interval-1
      templateType: Suspend
      deadline: 5m
```