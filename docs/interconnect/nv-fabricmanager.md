### Check NVIDIA Fabric Manager Status

**Purpose**

Verify that NVIDIA Fabric Manager (FM) is running properly and that the NVLink / NVSwitch fabric is successfully initialized on supported GPU platforms.

NVIDIA Fabric Manager is responsible for configuring and managing the NVSwitch fabric, coordinating GPU initialization, setting up GPU routing, and monitoring NVLink and NVSwitch errors.

It plays a critical role in enabling high-speed GPU-to-GPU communication on NVSwitch-based systems, such as NVIDIA HGX H100.

Abnormal Fabric Manager service status or unsuccessful fabric initialization may prevent GPUs from establishing NVLink P2P communication and negatively impact multi-GPU training workloads.

**Command**

```bash
# Check Fabric Manager service status
systemctl is-active nvidia-fabricmanager

# Check detailed service status
systemctl status nvidia-fabricmanager

# Check Fabric Manager logs from the current boot
journalctl -u nvidia-fabricmanager -b --no-pager

# Check Fabric Manager version
nv-fabricmanager --version

# Check GPU Fabric registration status
nvidia-smi -q -i 0 | grep -i -A 2 Fabric

# Check GPU-to-GPU NVLink topology
nvidia-smi topo -m

# Check NVLink link status
nvidia-smi nvlink -s
```

**Expected Result**

- NVIDIA Fabric Manager service reports `active` on platforms that require Fabric Manager.
- Fabric Manager version is compatible with the installed NVIDIA GPU driver.
- No unexpected Fabric Manager initialization failures are reported in service logs.
- All expected GPUs successfully register with the NVLink Fabric.
- GPU Fabric State reports `Completed`.
- GPU Fabric Status reports `Success`.
- NVLink and NVSwitch topology matches the expected platform architecture.
- No unexpected NVLink or NVSwitch errors are detected.

Example of a healthy GPU Fabric registration status:

```text
Fabric
    State                   : Completed
    Status                  : Success
```

**Abnormal Symptoms**

- `systemctl is-active nvidia-fabricmanager` returns `inactive`, `failed`, or `unknown` on a platform that requires the service.
- Fabric Manager fails to start or exits unexpectedly.
- Fabric Manager and NVIDIA GPU driver versions are incompatible.
- GPU Fabric State remains `In Progress` or does not reach `Completed`.
- GPU Fabric Status reports an error instead of `Success`.
- One or more GPUs fail to register with the NVLink Fabric.
- NVLink P2P communication is unavailable or incomplete.
- Fabric Manager logs report NVSwitch initialization, routing, or communication errors.
- NVLink or NVSwitch errors affect GPU-to-GPU communication.

**Troubleshooting**

- Check Fabric Manager service status using `systemctl status nvidia-fabricmanager`.
- Inspect Fabric Manager logs using `journalctl -u nvidia-fabricmanager -b` and `/var/log/fabricmanager.log`, if available.
- Verify Fabric Manager and NVIDIA GPU driver version compatibility.
- Check NVIDIA kernel driver and GPU availability using `nvidia-smi` and `lsmod`.
- Inspect GPU Fabric registration status using `nvidia-smi -q`.
- Check NVLink link status using `nvidia-smi nvlink -s`.
- Inspect GPU-to-GPU topology using `nvidia-smi topo -m`.
- Check NVSwitch-related kernel logs for SXid errors:

```bash
dmesg -T | grep -iE 'SXid|nvidia-nvswitch'
```

- Verify NVSwitch driver and related software components are installed and operating normally.
- On fourth-generation NVSwitch platforms, verify that the required NVLink Subnet Manager (NVLSM) components and services are operational.
- Compare Fabric Manager logs and GPU Fabric status with healthy nodes using the same hardware configuration.
- Collect NVIDIA diagnostic logs and contact the hardware vendor if Fabric initialization or NVLink errors persist.

**Note**

- Fabric Manager is required for applicable NVSwitch-based platforms, such as NVIDIA HGX H100 and HGX B200. It is generally not required for PCIe-only GPU servers without NVSwitch fabric.
- An `active` Fabric Manager service does not guarantee that all GPUs have successfully registered with the NVLink Fabric.
- On H100 and later platforms supporting Autonomous Link Initialization (ALI), NVLink link training is handled by the GPU and NVSwitch hardware. GPU registration with the NVLink Fabric is still required for NVLink P2P communication.
- Fabric Manager monitors NVLink and NVSwitch errors but does not automatically repair every physical link failure.
- Do not restart Fabric Manager or reset GPUs on production nodes without draining workloads and following the platform-specific recovery procedure.
- For fourth-generation NVSwitch platforms, additional NVLSM components are required for fabric initialization and routing.