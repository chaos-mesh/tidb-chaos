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
        - IO-Latency-2
        - Interval-2
        - IO-Latency-3
        - Interval-3
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
    - name: IO-Latency-2
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
    - name: Interval-2
      templateType: Suspend
      deadline: 5m
    - name: IO-Latency-3
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
    - name: Interval-3
      templateType: Suspend
      deadline: 5m
```

### TiDB Status
```
NAME                                     READY   STATUS        RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
advanced-tidb-discovery-9bcbcfb6-b6mwc   1/1     Running   0          52d     10.244.3.33   tikv-2   <none>           <none>
advanced-tidb-discovery-9bcbcfb6-jdd8r   1/1     Running   0          5d21h   10.244.2.40   tikv-1   <none>           <none>
advanced-tidb-pd-0                       1/1     Running   0          51d     10.244.7.23   pd-3     <none>           <none>
advanced-tidb-pd-1                       1/1     Running   0          51d     10.244.5.27   pd-1     <none>           <none>
advanced-tidb-pd-2                       1/1     Running   1          51d     10.244.6.25   pd-2     <none>           <none>
advanced-tidb-tidb-0                     2/2     Running   0          47d     10.244.0.33   tidb-1   <none>           <none>
advanced-tidb-tidb-1                     2/2     Running   0          51d     10.244.1.32   tidb-2   <none>           <none>
advanced-tidb-tidb-2                     2/2     Running   0          51d     10.244.1.31   tidb-2   <none>           <none>
advanced-tidb-tikv-0                     3/3     Running   0          51d     10.244.2.35   tikv-1   <none>           <none>
advanced-tidb-tikv-1                     3/3     Running   0          51d     10.244.3.35   tikv-2   <none>           <none>
advanced-tidb-tikv-2                     3/3     Running   0          51d     10.244.4.33   tikv-3   <none>           <none>
monitor-monitor-0                        3/3     Running   0          52d     10.244.1.27   tidb-2   <none>           <none>
```