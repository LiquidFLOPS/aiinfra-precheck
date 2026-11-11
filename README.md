# AI Infra Precheck

A practical, operations-driven checklist for validating GPU cluster infrastructure before running distributed AI training workloads.

AI Infra Precheck helps GPU cluster operators identify infrastructure issues that may cause NCCL test failures, low communication bandwidth, GPU health problems, and slow training nodes.

## What Does It Cover?

| Area | Checks |
|---|---|
| GPU Health | GPU availability,GPU Persistence Mode,Xid errors, ECC errors, GPU Performance State and Throttling |
| Interconnect | GPU Topology,P2P, NVLink, PCIe topology |
| Network | InfiniBand, RoCE, RDMA, GPUDirect RDMA |
| PCIe | GPU/NIC PCIe link speed, width, ACS,topology degradation |
| Performance | NCCL Tests, RDMA bandwidth, communication bottlenecks |
| System | CPU, NUMA, memory, storage and power management |

## Who Is This For?

- AI Infrastructure / GPU Cluster SREs
- HPC System Administrators
- GPU Infrastructure Engineers
- Engineers troubleshooting distributed training performance

## Project Scope

This is a documentation-driven operational checklist based on real-world GPU cluster operations. It is not an automated diagnostic tool.

Commands that modify system or network configuration are explicitly marked. Always verify hardware compatibility and your platform's operational requirements before applying remediation steps.

## Contributions

Contributions, corrections, and additional checks are welcome!

Have a useful GPU cluster troubleshooting command or real-world diagnostic experience? Feel free to open an issue or submit a pull request.