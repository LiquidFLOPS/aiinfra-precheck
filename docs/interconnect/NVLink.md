### Check NVLink Link Status and Bandwidth

**Purpose**

Verify the operational status, link count, and bandwidth of NVLink high-speed interconnect links between GPUs.

Ensure that all expected NVLink links are active and operating at their configured bandwidth.

Abnormal NVLink link status or bandwidth degradation may reduce GPU-to-GPU communication performance and negatively impact multi-GPU training workloads.

**Command**

```bash
# Check NVLink link status and bandwidth
nvidia-smi nvlink -s

# Check NVLink error counters
nvidia-smi nvlink -e

# Check GPU NVLink connectivity topology
nvidia-smi topo -m

# Check NVLink capabilities
nvidia-smi nvlink -c
```

**Expected Result**

For H100 HGX / NVSwitch platforms designed for full NVLink connectivity:

- Each GPU should have 18 expected NVLink links.
- All expected NVLink links should be active.
- Each NVLink link should report approximately 26.562 GB/s bandwidth.
- NVLink link status should be consistent across all GPUs.
- No unexpected NVLink link degradation or recurring link errors should be detected.

Example of a healthy H100 NVLink configuration:

```text
GPU 0: NVIDIA H100 80GB HBM3 (UUID: GPU-XXXXXXXX)

    Link 0:  26.562 GB/s
    Link 1:  26.562 GB/s
    Link 2:  26.562 GB/s
    Link 3:  26.562 GB/s
    Link 4:  26.562 GB/s
    Link 5:  26.562 GB/s
    Link 6:  26.562 GB/s
    Link 7:  26.562 GB/s
    Link 8:  26.562 GB/s
    Link 9:  26.562 GB/s
    Link 10: 26.562 GB/s
    Link 11: 26.562 GB/s
    Link 12: 26.562 GB/s
    Link 13: 26.562 GB/s
    Link 14: 26.562 GB/s
    Link 15: 26.562 GB/s
    Link 16: 26.562 GB/s
    Link 17: 26.562 GB/s
```

**Abnormal Symptoms**

- One or more expected NVLink links are missing.
- NVLink links are reported as `Inactive` unexpectedly.
- NVLink bandwidth is lower than the expected platform specification.
- NVLink link status or bandwidth is inconsistent across GPUs.
- NVLink Replay Errors, Recovery Errors, or CRC Errors continue to increase.
- GPU-to-GPU communication performance is degraded due to NVLink failures.
- NVLink connectivity is incomplete or inconsistent with the expected GPU topology.

**Troubleshooting**

- Check NVLink link status and bandwidth using `nvidia-smi nvlink -s`.
- Check NVLink error counters using `nvidia-smi nvlink -e`.
- Identify the affected GPU and NVLink index.
- Inspect GPU-to-GPU connectivity using `nvidia-smi topo -m`.
- Verify NVSwitch and Fabric Manager status on NVSwitch-based platforms.
- Inspect Fabric Manager logs for NVLink initialization and configuration failures.
- Check NVLink and NVSwitch hardware health.
- Verify NVIDIA driver and GPU firmware compatibility.
- Compare NVLink link count, bandwidth, and error counters with healthy GPUs or nodes.
- Collect NVIDIA diagnostic logs and contact the hardware vendor if link failures persist.

**Note**

- The expected NVLink link count and bandwidth depend on GPU architecture and server platform configuration.
- For H100 HGX platforms, 18 NVLink links per GPU and approximately 26.562 GB/s per link are expected.
- `nvidia-smi nvlink -s` reports link status and configured bandwidth, not actual application-level communication throughput.
- NVLink error counters may be cumulative. Monitor counter increments over time rather than treating every non-zero historical value as a hardware failure.
- A link reported as `Sleep` is not necessarily faulty. Evaluate its state according to platform power management and workload conditions.
- Validate actual GPU-to-GPU communication performance using NCCL Tests or nvbandwidth.