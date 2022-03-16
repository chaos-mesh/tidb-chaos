# Test IO Chaos on TiDB
## IO Latency

### Description
Test the availability of TiDB cluster in IO Chaos with IO latency scenarios.

### Hypothesis

When about one third of TiKV pod in TiDB cluster suffered a high IO latency , the TiDB cluster can still provide services normally.

### Preparation

1. Install chaos-mesh
2. A TiDB cluster on k8s with at least 3 tikv POD.
3. Runing payload on TiDB cluster

### Quick start
1. Chaos Mesh fault YAML configuration:
```
IOChaos-Latency.yaml
```
```
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
  percent: 100
  volumePath: /var/lib/tikv
  containerName: tikv
```
2. Using Kubectl to create the experiment:
```
kubectl create -f IOChaos-Latency.yaml
```
3. Verifying TiDB's status:

    Check QPS & TPS of TiDB in Grafana.
4. Resault:

Judge whether the hypothesis is correct or not based on the results of the test process.

### More example

In practise, we found TiDB or other softwares can not handle chaos situation with a short intervals, like 3 min IO latency with 3 min no error continue for an hour. It is really hard to keep working when the problem happen.
(This also happens in Network Chaos)


Continues IO Latency intervals:
```
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
        percent: 50
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
        percent: 50
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
        percent: 50
        methods:
          - read
          - write
    - name: Interval-3
      templateType: Suspend
      deadline: 5m
```
