### Check GPU Device Count

**Purpose**

Verify that all expected GPU devices are detected and available on the node.

**Command**

```bash
nvidia-smi -L | wc -l
```

**Expected Result**

The command should return `8` for an 8-GPU node.

The detected GPU count must match the expected GPU configuration.

**Abnormal Symptoms**

- GPU count is less than 8 (e.g. 7, 6, or fewer GPUs detected).
- GPU count is 0.
- GPU devices are missing or not detected by the NVIDIA driver.

**Troubleshooting**

- Check GPU enumeration using `nvidia-smi -L` and `nvidia-smi`.
- Check NVIDIA driver status and initialization errors.
- Inspect kernel logs for GPU XID errors and NVRM errors.
- Check PCIe device enumeration using `lspci | grep -i nvidia`.
- Check GPU power connections, PCIe slots, and physical hardware.
- Compare the detected GPU count with healthy nodes.