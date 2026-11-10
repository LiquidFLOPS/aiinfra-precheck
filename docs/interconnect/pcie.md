### Check PCIe ACS Configuration

**Purpose**

Verify PCIe Access Control Services (ACS) configuration to ensure that PCIe peer-to-peer (P2P) communication between GPUs and RDMA NICs works as expected.

Incorrect ACS redirection settings may redirect PCIe P2P traffic through the CPU root complex instead of allowing direct peer-to-peer communication, potentially causing reduced GPUDirect RDMA performance or communication failures.

**Command**

```bash
# Check ACS configuration on PCIe devices
sudo lspci -D -vvv | grep -E '^[[:xdigit:]]{4}:|ACSCap:|ACSCtl:'

# Inspect the full ACS configuration of a specific PCIe bridge
sudo lspci -D -s <PCI_BDF> -vvv

# Check PCIe topology
sudo lspci -tv

# Check GPU topology
nvidia-smi topo -m
```

**Expected Result**

- ACS configuration on PCIe bridges matches the platform-specific hardware and GPU Direct requirements.
- No unexpected PCIe P2P Request Redirect or Completion Redirect on bridges where direct P2P communication is required.
- GPU-to-GPU and GPU-to-NIC PCIe communication paths comply with the platform design.
- IOMMU configuration matches the requirements of the target GPU P2P / GPUDirect RDMA configuration.

Pay attention to the following ACS control flags in `lspci -vvv` output:

- `ReqRedir+`: ACS P2P Request Redirect is enabled.
- `CmpltRedir+`: ACS P2P Completion Redirect is enabled.
- `UpstreamFwd+`: ACS Upstream Forwarding is enabled.
- `SrcValid+`: ACS Source Validation is enabled.

Note that `SrcValid+` alone does not necessarily indicate that P2P traffic is being redirected. Inspect the complete ACS configuration and compare it against the platform-specific requirements.

**Abnormal Symptoms**

- Unexpected ACS Request Redirect or Completion Redirect is enabled on PCIe bridges along the GPU-to-NIC or GPU-to-GPU communication path.
- GPU P2P communication is unavailable or performs below expectations.
- GPUDirect RDMA traffic is redirected through the CPU root complex.
- NCCL communication experiences low bandwidth, performance degradation, or hangs.
- ACS configuration differs from the recommended BIOS or platform configuration.

**Troubleshooting**

- Inspect the PCIe topology using `lspci -tv` and `nvidia-smi topo -m` to identify the communication paths between GPUs and RDMA NICs.
- Check ACS Capability and Control registers on the relevant PCIe bridges using `lspci -vvv`.
- Inspect IOMMU configuration using the following commands:

```bash
cat /proc/cmdline
dmesg | grep -iE 'iommu|dmar|default domain'
```

- Verify BIOS settings related to PCIe ACS, VT-d, and IOMMU according to the platform vendor's recommendations.
- For Broadcom PLX PCIe switches, check whether ACS can be configured through the OS using `setpci`, following the hardware vendor's instructions.
- If ACS configuration is modified through `setpci`, verify the changes and ensure they persist after system reboot.
- Validate GPU P2P connectivity using `nvidia-smi topo -p2p p`.
- Validate GPUDirect RDMA and NCCL communication performance after configuration changes.
- Compare ACS settings and PCIe topology with healthy nodes.

**Note**

- Do not blindly disable all ACS features across PCIe devices. Some platforms require specific ACS bits to remain enabled.
- Disabling ACS may affect PCIe device isolation and virtualization security.
- Virtualized environments may require ACS for device isolation. Do not disable ACS on virtual machines without checking the platform requirements.
- IOMMU and ACS are separate PCIe-related mechanisms and should be checked independently.