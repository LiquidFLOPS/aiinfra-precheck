### Check GPU Driver Status

**Purpose**

Verify that the NVIDIA GPU driver is properly loaded and initialized, and that all GPU devices are accessible through the driver.

**Command**

```bash
# Check NVIDIA driver version
cat /proc/driver/nvidia/version

# Check NVIDIA kernel modules
lsmod | grep '^nvidia'

# Check GPU accessibility and driver status
nvidia-smi

# Check NVIDIA driver initialization errors
journalctl -k --since "24 hours ago" --no-pager \
  | grep -iE 'NVRM|nvidia.*(failed|error|failure)'
```

**Expected Result**

- NVIDIA kernel modules are successfully loaded.
- NVIDIA driver version is correctly reported.
- `nvidia-smi` executes successfully and detects all expected GPU devices.
- No unexpected NVIDIA driver loading or initialization errors are reported in kernel logs.

**Abnormal Symptoms**

- NVIDIA kernel modules are missing or fail to load.
- `nvidia-smi` reports `NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver`.
- `nvidia-smi` reports `Failed to initialize NVML`.
- GPU devices are detected by PCIe but are not accessible through the NVIDIA driver.
- NVIDIA driver initialization failures appear in kernel logs.
- Driver API version mismatch or kernel module version mismatch occurs.
- NVIDIA driver fails to load after a kernel upgrade or system reboot.

**Troubleshooting**

- Check NVIDIA kernel module status using `lsmod` and `modinfo nvidia`.
- Inspect kernel logs using `journalctl -k` and `dmesg` for NVIDIA driver loading and initialization errors.
- Verify NVIDIA driver installation and version using `nvidia-smi` and `/proc/driver/nvidia/version`.
- Check whether the NVIDIA kernel module matches the running kernel version.
- If the driver module fails to load, use `dkms status` to verify the DKMS compilation status and kernel headers.
- Check Secure Boot and kernel module signing if module loading is rejected.
- Check for NVIDIA driver version mismatches between the kernel module and user-space libraries.
- Verify GPU PCIe enumeration using `lspci`.
- Reinstall or reload the NVIDIA driver only after confirming the root cause and coordinating with workload owners.

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

### Check GPU XID Errors

**Purpose**

Detect GPU-related errors reported by the NVIDIA driver and identify potential application, GPU hardware, PCIe, NVLink, ECC, and driver/firmware issues.

**Command**

```bash
journalctl -k --since "24 hours ago" --no-pager \
  | grep -E 'NVRM: Xid'
```

Alternatively, check kernel logs:

```bash
dmesg -T | grep -i 'NVRM: Xid'
```

**Expected Result**

No unexpected GPU XID errors should be reported during the inspection period.

Pay attention to the following XID errors:

| XID | Description | Recommended Handling |
|---|---|---|
| 13 | Graphics Engine Exception | Usually application-related. Investigate CUDA application errors. |
| 31 | GPU Memory Page Fault | Usually caused by illegal memory access. Check application code and CUDA errors. |
| 32 | Invalid or corrupted push buffer stream | Check PCIe link quality and hardware connectivity. |
| 38 | Driver firmware error | Investigate NVIDIA driver and firmware compatibility. |
| 43 | GPU stopped processing | Usually a software-induced application fault. GPU generally remains healthy. |
| 45 | Preemptive channel removal | Often caused by application termination or previous GPU errors. Check preceding XID events. |
| 48 | Double Bit ECC Error (DBE) | Uncorrectable ECC error. GPU reset or node reboot may be required. |
| 61 | Internal micro-controller breakpoint/warning | Verify GPU architecture and driver-specific XID documentation. |
| 62 | Internal micro-controller halt | Investigate GPU internal errors. GPU reset may be required. |
| 63 | ECC page retirement or row remapping event | Check GPU memory error and row remapping status. |
| 64 | ECC page retirement or row remapping failure | Investigate GPU memory errors. GPU reset and hardware diagnostics may be required. |
| 68 | NVDEC exception | Investigate the affected application and video decoding engine. |
| 74 | NVLink error | Check NVLink connections, NVSwitch, and remote GPU health. |
| 79 | GPU has fallen off the bus | Investigate PCIe connectivity, GPU hardware, power, and driver issues. |
| 92 | High single-bit ECC error rate | Monitor ECC error counters and investigate potential hardware issues. |
| 94 | Contained memory error | Error is isolated to one application. Restart the affected application. |
| 95 | Uncontained memory error | Multiple applications may be affected. GPU reset is required before restarting workloads. |
| 120 | GSP error | Investigate GSP firmware/driver errors. GPU reset or node power cycle may be required. |

**Abnormal Symptoms**

- Unexpected XID errors are reported in kernel logs.
- Repeated XID errors occur on the same GPU.
- GPU becomes unavailable or inaccessible.
- ECC errors or GPU memory remapping failures are detected.
- PCIe or NVLink connectivity errors affect GPU communication.
- GPU workloads terminate unexpectedly or experience performance degradation.

**Troubleshooting**

For application-related XID errors (13, 31, 43, 45, 68):

1. Resubmit the workload and observe whether the XID errors disappear.
2. If the errors persist, inspect application logs and CUDA code to identify potential user-code issues.
3. Use CUDA debugging tools such as Compute Sanitizer or cuda-gdb when necessary.
4. If application code is confirmed to be correct, contact the operations team for further investigation.

For hardware, interconnect, ECC, and driver-related XID errors:

- **XID 32:** Inspect PCIe link speed, width, PCIe AER logs, slots, and risers.
- **XID 38:** Check NVIDIA driver and GPU firmware compatibility.
- **XID 48:** Check ECC error counters using `nvidia-smi -q -d ECC`. GPU reset or node reboot may be required.
- **XID 61/62:** Inspect GPU internal errors and NVIDIA driver logs. Follow the XID handling guidance for the specific GPU and driver version.
- **XID 63/64:** Check ECC page retirement or row remapping status using `nvidia-smi -q`.
- **XID 74:** Inspect NVLink status using `nvidia-smi nvlink -s`. Check the connected GPU and NVSwitch health.
- **XID 79:** Inspect PCIe AER logs, GPU PCIe connectivity, power connections, slots, and risers.
- **XID 92:** Monitor single-bit ECC error counters and investigate repeated errors.
- **XID 94/95:** Identify affected applications. Restart the affected application for XID 94; perform a GPU reset before restarting workloads for XID 95.
- **XID 120:** Check NVIDIA driver and GSP-related logs. Consider GPU reset or node power cycle if the error persists.

If critical errors persist or hardware failure is suspected:

1. Drain the affected node from new workloads.
2. Coordinate with workload owners before stopping or resetting GPU processes.
3. Collect kernel logs, GPU diagnostics, and hardware information.
4. Contact the operations team or hardware vendor for further investigation and repair.

**Note**

XID severity and recovery behavior may vary depending on GPU architecture, driver version, and firmware version. Always refer to the corresponding NVIDIA XID documentation before performing recovery operations.

References:
- https://docs.nvidia.com/deploy/xid-errors/latest/index.html
- https://docs.nvidia.com/deploy/gpu-debug-guidelines/index.html

### Check GPU Performance State and Throttling

**Purpose**

Detect whether GPUs experience performance degradation or clock throttling due to high temperature, power limits, external power issues, or other hardware protection mechanisms.

GPU throttling may reduce GPU operating clocks and directly impact model training performance.

**Command**

```bash
# Check GPU performance state and clock event reasons
nvidia-smi -q -d PERFORMANCE

# Check GPU temperature, power consumption, and power limits
nvidia-smi -q -d TEMPERATURE,POWER

# Query key GPU performance metrics
nvidia-smi --query-gpu=index,pstate,temperature.gpu,power.draw,power.limit,utilization.gpu --format=csv
```

**Expected Result**

- GPU Performance State should be consistent with the current workload. P0 is typically expected under intensive GPU workloads.
- No unexpected hardware or software thermal slowdown should be active.
- No unexpected hardware power brake events should be detected.
- GPU temperature should remain within the recommended operating range.
- GPU power consumption and clock frequencies should be consistent with the workload and configured power limits.

Pay attention to the following Clocks Event Reasons:

| Event Reason | Description |
|---|---|
| Idle | GPU is idle and clocks are reduced. Normal when no workload is running. |
| SW Power Cap | GPU clocks are limited by the configured software power limit. |
| HW Slowdown | Hardware slowdown is engaged due to thermal or power protection. |
| HW Thermal Slowdown | GPU clocks are reduced due to excessive temperature. |
| HW Power Brake Slowdown | External power brake is triggered, for example by the system power supply. |
| SW Thermal Slowdown | Software thermal management reduces GPU clocks due to high temperature. |
| Sync Boost | GPU clocks are limited by the lowest clock frequency within a Sync Boost group. |

**Abnormal Symptoms**

- HW Thermal Slowdown is `Active`.
- SW Thermal Slowdown is `Active` during intensive workloads.
- HW Power Brake Slowdown is `Active`.
- HW Slowdown is `Active`, indicating hardware protection is reducing GPU clocks.
- SW Power Cap is unexpectedly active and limits GPU performance.
- GPU remains in a low-performance state during intensive training workloads.
- GPU temperature approaches or exceeds the maximum operating temperature.
- GPU clock frequencies are significantly lower than expected under comparable workloads.

**Troubleshooting**

- Check GPU performance state, clock event reasons, and throttling counters using `nvidia-smi -q -d PERFORMANCE`.
- Check GPU temperature, fan status, and power consumption using `nvidia-smi -q -d TEMPERATURE,POWER`.
- For thermal slowdown, inspect server cooling systems, airflow, fan operation, heatsinks, and rack inlet temperature.
- For power limit issues, compare actual power consumption against the configured GPU power limit.
- For hardware power brake events, inspect server power supplies, power connections, and system power delivery.
- Check GPU application clock settings and Sync Boost configuration.
- Compare GPU clock frequencies, temperatures, and power consumption with healthy nodes running similar workloads.
- Investigate recurring throttling events and collect NVIDIA diagnostic logs if hardware or cooling failures are suspected.

**Note**

- `Idle: Active` is normal when no GPU workload is running and should not be treated as a throttling failure.
- `SW Power Cap: Active` does not necessarily indicate a hardware fault. It may simply mean the GPU has reached its configured power limit.
- Evaluate throttling status under actual training workloads rather than relying only on idle-state snapshots.
- Performance State and clock event reasons may vary depending on GPU architecture, driver version, workload, and power configuration.