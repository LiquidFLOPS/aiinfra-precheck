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

### Check GPU Persistence Mode

**Purpose**

Verify that GPU Persistence Mode is enabled to keep the NVIDIA driver loaded and reduce GPU initialization overhead.

**Command**

```bash
nvidia-smi --query-gpu=persistence_mode --format=csv,noheader
```

**Expected Result**

All GPUs should report `Enabled`.

The Persistence Mode status should be consistent across all GPUs and match the cluster configuration baseline.

**Abnormal Symptoms**

- Persistence Mode is `Disabled` on one or more GPUs.
- Persistence Mode status is inconsistent across GPUs.

**Troubleshooting**

- Check Persistence Mode status using `nvidia-smi -q`.
- Enable Persistence Mode using `sudo nvidia-smi -pm 1`.
- Check NVIDIA driver status and initialization errors.
- Check whether `nvidia-persistenced` is running, if used by the platform.
- Verify Persistence Mode configuration after node reboot.