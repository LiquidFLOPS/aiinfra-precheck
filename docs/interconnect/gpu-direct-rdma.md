### Check GPUDirect RDMA (GDR) Configuration and Status

**Purpose**

Verify whether GPUDirect RDMA (GDR) is properly configured and available for direct data transfers between GPU memory and RDMA network devices.

GPUDirect RDMA enables compatible network adapters to directly access GPU memory without staging data through host memory, reducing CPU overhead and communication latency.

Validate the required GPU drivers, RDMA drivers, peer memory modules, DMA-BUF support, and GPU-to-NIC PCIe topology.

**Command**

```bash
# Check whether the NVIDIA peer memory module is loaded
lsmod | grep -E 'nvidia_peermem|nv_peer_mem'

# Check whether the NVIDIA peer memory module is available
modinfo nvidia-peermem

# Check RDMA devices and link status
rdma link
ibdev2netdev

# Check GPU-to-NIC PCIe topology
nvidia-smi topo -m
lspci -tv

# Check kernel version
uname -r

# Check IOMMU configuration
cat /proc/cmdline
dmesg | grep -iE 'iommu|dmar|default domain'
```

**Expected Result**

- Compatible GPU and RDMA NIC drivers are installed and operating normally.
- At least one supported GPUDirect RDMA implementation is available:
  - Legacy peer memory: `nvidia-peermem` is successfully loaded.
  - DMA-BUF: The NVIDIA Open GPU Kernel Module, Linux kernel, CUDA driver, and RDMA NIC driver meet the DMA-BUF requirements.
- RDMA devices are correctly detected and their links are active.
- GPU-to-NIC PCIe topology matches the expected platform architecture.
- No unexpected PCIe ACS or IOMMU configuration prevents GPU-to-NIC P2P communication.
- GPUDirect RDMA functionality is successfully validated using GPU-memory-based RDMA or NCCL communication tests.

**Abnormal Symptoms**

- `nvidia-peermem` is required by the legacy configuration but is not loaded.
- `nvidia-peermem` fails to load due to missing kernel symbols, incompatible RDMA drivers, or module compilation errors.
- Neither the legacy peer memory path nor the DMA-BUF path is available.
- RDMA NIC devices are missing or their ports are not active.
- GPU and NIC PCIe topology does not meet the platform's GPUDirect RDMA requirements.
- PCIe ACS or translated IOMMU configuration interferes with GPU-to-NIC P2P communication.
- GPUDirect RDMA initialization fails.
- NCCL cannot establish GPU Direct RDMA communication and falls back to host-memory-based transfers.
- GPU-memory-based RDMA tests fail or show significantly degraded performance.

**Troubleshooting**

- Check NVIDIA peer memory module status using `lsmod`.
- Verify module availability and dependencies using `modinfo nvidia-peermem`.
- For legacy peer memory configurations, load the module using:

```bash
sudo modprobe nvidia-peermem
```

- Inspect kernel logs for peer memory initialization failures:

```bash
dmesg -T | grep -iE 'nvidia_peermem|nv_peer_mem|peer memory'
```

- Check whether the RDMA driver stack is correctly installed and supports peer memory registration.
- Verify DMA-BUF prerequisites, including the NVIDIA Open GPU Kernel Module, Linux kernel version, CUDA driver, and RDMA NIC driver compatibility.
- Inspect GPU-to-NIC PCIe connectivity using `nvidia-smi topo -m` and `lspci -tv`.
- Check PCIe ACS and IOMMU configurations for potential P2P communication restrictions.
- Verify RDMA NIC health and port connectivity using `rdma link`, `ibstat`, and `ibdev2netdev`.
- Validate GPU-memory-based RDMA transfers using a CUDA-aware version of perftest with GPUDirect RDMA support.
- Enable NCCL debug logging and inspect network initialization and GDR-related messages:

```bash
NCCL_DEBUG=INFO \
NCCL_DEBUG_SUBSYS=INIT,NET \
./build/all_reduce_perf -b 8M -e 1G -f 2 -g 8
```

- Compare GPUDirect RDMA configuration and communication performance with healthy nodes using the same hardware and software stack.

**Note**

- GPUDirect RDMA can use either the legacy `nvidia-peermem` kernel module or the Linux DMA-BUF mechanism.
- The absence of `nvidia-peermem` does not necessarily indicate a GDR failure when the DMA-BUF implementation is supported and enabled.
- DMA-BUF support requires compatible GPU drivers, kernel versions, and RDMA NIC drivers. Verify the prerequisites for the specific software stack.
- `NCCL_DMABUF_ENABLE` controls DMA-BUF registration in NCCL and defaults to `1`. NCCL automatically disables this feature when the required components are unsupported.
- Loading `nvidia-peermem` alone does not prove that GPUDirect RDMA is working. Actual GPU-memory-based RDMA or NCCL communication tests are required.
- Ordinary RDMA bandwidth tests using host memory do not validate GPUDirect RDMA functionality.
- Validate GPU-to-NIC PCIe topology and platform-specific ACS/IOMMU requirements before enabling GPUDirect RDMA.