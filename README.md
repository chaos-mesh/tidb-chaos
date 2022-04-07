# tidb-chaos

This repository is used to record the method, process, and results of chaos engineering testing of TiDB by using Chaos Mesh.

## Test Environment

- Linux amd64
- TiDB 5.4
- Kubernetes 1.21

For details about the deployment and architecture of the TIDB cluster, see [Deployment](./deployment/deployment.md).


## Format

The record for each test scenario should contain the content below:

- Description

- Hypothesis

- Process

- Result

See [fotmat of record](./format.md) for details.

## Test Scenarios

### Network

- [Network Delay](./network/delay.md)
- [Network Bandwidth](./network/bandwidth.md)
- [Network Partition](./network/partition.md)
- [Network Packet Loss](./network/loss.md)
- Network Packet Reorder
- Network Packet Duplicate
- Network Packet Corrupt

### IO

- IO Latency
- IO Fault

### Stress

- CPU Stress
- Memory Stress

### Kill Server

- Kill server(PD, TiKV or TiDB)

