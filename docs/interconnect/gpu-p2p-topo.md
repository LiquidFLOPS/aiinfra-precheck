### Check GPU P2P Topology State

**Purpose**

Verify whether Peer-to-Peer (P2P) communication between GPUs is supported and available.

Check the P2P read access capability between GPU pairs to ensure that GPUs can directly access each other's memory without relying on CPU-mediated memory transfers.

For systems designed to support full GPU-to-GPU P2P communication, all GPU pairs within the same node should ideally report `OK`.

**Command**

```bash
# Check GPU-to-GPU P2P read capability
nvidia-smi topo -p2p r

# Check GPU-to-GPU P2P write capability
nvidia-smi topo -p2p w

# Check NVLink P2P capability
nvidia-smi topo -p2p n

# Inspect GPU interconnect topology
nvidia-smi topo -m
```

**Expected Result**

- All GPU pairs expected to support P2P read access should report `OK`.
- The P2P capability matrix should match the expected GPU interconnect architecture.
- On an 8-GPU node designed for full GPU-to-GPU P2P connectivity, all off-diagonal entries should report `OK`.
- No unexpected unsupported or unknown P2P connections should be detected.

Example of a healthy 8-GPU P2P read capability matrix:

```text
        GPU0    GPU1    GPU2    GPU3    GPU4    GPU5    GPU6    GPU7
GPU0    X       OK      OK      OK      OK      OK      OK      OK
GPU1    OK      X       OK      OK      OK      OK      OK      OK
GPU2    OK      OK      X       OK      OK      OK      OK      OK
GPU3    OK      OK      OK      X       OK      OK      OK      OK
GPU4    OK      OK      OK      OK      X       OK      OK      OK
GPU5    OK      OK      OK      OK      OK      X       OK      OK
GPU6    OK      OK      OK      OK      OK      OK      X       OK
GPU7    OK      OK      OK      OK      OK      OK      OK      X

Legend:

  X    = Self
  OK   = Status Ok
  CNS  = Chipset not supported
  GNS  = GPU not supported
  TNS  = Topology not supported
  NS   = Not supported
  U    = Unknown
```

**Abnormal Symptoms**

- One or more GPU pairs report `NS` instead of `OK`.
- `CNS` indicates that the chipset does not support the requested P2P capability.
- `GNS` indicates that the GPU does not support the requested P2P capability.
- `TNS` indicates that the PCIe or GPU topology does not support the requested P2P capability.
- `U` indicates that the P2P capability status is unknown.
- P2P capability is inconsistent across GPU pairs compared with the expected hardware topology.
- GPU-to-GPU P2P communication is unavailable or NCCL falls back to alternative communication paths.

**Troubleshooting**

- Inspect GPU interconnect topology using `nvidia-smi topo -m`.
- Check P2P read and write capabilities using `nvidia-smi topo -p2p r` and `nvidia-smi topo -p2p w`.
- For NVLink-based systems, verify NVLink P2P capability using `nvidia-smi topo -p2p n`.
- Verify that GPU models, driver versions, and platform configurations support the expected P2P capabilities.
- Inspect PCIe topology, link speed, and link width using `lspci -tv` and `lspci -vv`.
- Check PCIe ACS and IOMMU configurations for potential P2P communication restrictions.
- Verify NVLink and NVSwitch status on systems using NVSwitch fabric.
- For virtualized environments, check GPU passthrough configuration and PCIe topology exposure.
- Use CUDA `simpleP2P` to verify actual GPU-to-GPU memory access and data transfer.
- Use NVIDIA `nvbandwidth` or NCCL Tests to evaluate actual GPU-to-GPU communication bandwidth.
- Compare P2P capability matrices with healthy nodes using the same hardware configuration.

**Note**

- `nvidia-smi topo -p2p r` checks P2P read capability only. It does not guarantee that all P2P operations or actual data transfers will work correctly.
- `OK` indicates that the requested P2P capability is supported by the GPU pair. It does not guarantee expected communication bandwidth.
- Not all GPU platforms are designed to support full P2P connectivity between every GPU pair. Evaluate results against the expected platform topology.
- A successful P2P capability check should be followed by actual communication tests when validating GPU cluster performance.