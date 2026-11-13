# InfiniBand Health Check

### Check InfiniBand NIC Detection and Port Status

**Purpose**

Verify that all expected InfiniBand network adapters are correctly detected by the operating system and that their ports are operational.

Ensure that RDMA devices, InfiniBand ports, and associated network interfaces are correctly initialized and available for distributed GPU communication.

**Command**

```bash
# List detected RDMA devices
ibv_devices

# Display InfiniBand device and port information
ibstat

# Display detailed RDMA device information
ibv_devinfo -v

# Check RDMA device and link status
rdma link

# Map RDMA devices to network interfaces
ibdev2netdev

# Check NIC PCIe device enumeration
lspci -Dnn | grep -iE 'Mellanox|NVIDIA'
```

**Expected Result**

- All expected InfiniBand NICs are detected by the operating system.
- RDMA devices are correctly registered and available.
- All configured InfiniBand ports report `State: Active`.
- Physical Port State reports `LinkUp`.
- Link Layer reports `InfiniBand`.
- All expected NIC ports are correctly mapped to RDMA devices and network interfaces.
- The number of detected RDMA devices and ports matches the server hardware configuration.

**Abnormal Symptoms**

- One or more expected InfiniBand NICs are missing from `ibv_devices`.
- RDMA devices fail to initialize or register.
- IB Port State reports `Down`, `Init`, or another unexpected state instead of `Active`.
- Physical State reports `Polling`, `Disabled`, or another unexpected state instead of `LinkUp`.
- InfiniBand ports are not correctly mapped to network interfaces.
- The number of detected RDMA devices differs from the expected hardware configuration.

**Troubleshooting**

- Check PCIe device enumeration using `lspci -Dnn`.
- Verify NVIDIA RDMA driver modules using `lsmod | grep mlx5`.
- Check RDMA device registration using `ibv_devices` and `rdma dev`.
- Inspect port status using `ibstat` and `ibv_devinfo -v`.
- Check NVIDIA OFED driver and firmware compatibility.
- Inspect PCIe AER logs and kernel messages for NIC initialization failures.
- Check NIC power connections, PCIe slots, and risers.
- Compare RDMA device enumeration and port status with healthy nodes.

---

### Check InfiniBand Port State, Physical State, Speed and Width

**Purpose**

Verify the operational state, physical link state, link speed, and link width of InfiniBand ports.

Ensure that all configured InfiniBand ports establish stable links at their expected speed and width.

Abnormal link states, speed downgrades, or width degradation may reduce RDMA communication bandwidth and negatively impact multi-node NCCL training performance.

**Command**

```bash
# Check InfiniBand port status
ibstat

# Check detailed active port speed and width
ibv_devinfo -v | grep -E 'hca_id:|port:|state:|active_width:|active_speed:|phys_state:|link_layer:'

# Check RDMA link status
rdma link show

# Check InfiniBand port information
ibstatus
```

**Expected Result**

- IB Port State should report `Active`.
- Physical State should report `LinkUp`.
- Active Link Speed should match the expected InfiniBand generation and platform specification.
- Active Link Width should match the expected port width, typically `4X` for ConnectX adapters.
- All configured ports should operate at their expected link speed and width.
- No unexpected link degradation should be detected.

Example of a healthy InfiniBand port:

```text
CA 'mlx5_0'
    Port 1:
        State: Active
        Physical state: LinkUp
        Rate: 400
        Link layer: InfiniBand
```

Example of detailed port information:

```text
hca_id: mlx5_0
    port: 1
        state: PORT_ACTIVE (4)
        active_width: 4X (2)
        active_speed: 100.0 Gbps
        phys_state: LINK_UP (5)
        link_layer: InfiniBand
```

**Abnormal Symptoms**

- IB Port State is not `Active`.
- Physical State is not `LinkUp`.
- Active Link Speed is lower than the expected platform specification.
- Active Link Width is lower than the expected width, such as `1X` or `2X` instead of `4X`.
- Different ports on identical NICs report inconsistent link speeds or widths.
- InfiniBand ports frequently transition between `Active` and `Down`.
- RDMA communication bandwidth is significantly lower than expected.

**Troubleshooting**

- Check port status and link configuration using `ibstat` and `ibv_devinfo -v`.
- Inspect port physical state and link transitions using `rdma link show`.
- Check InfiniBand switch port configuration and negotiated link speed.
- Inspect cable connections, optical transceivers, and port health.
- Check NIC firmware and driver compatibility.
- Inspect IB port error counters using `perfquery` and `ibqueryerrors`.
- Compare link speed and width with healthy nodes using the same NIC model and switch configuration.
- Run RDMA bandwidth tests using `ib_write_bw` or `ib_read_bw` to validate actual network performance.

---

### Check InfiniBand Port Flapping

**Purpose**

Detect unstable InfiniBand links that repeatedly transition between active and down states.

Frequent link flapping may indicate faulty cables, optical transceivers, switch ports, NIC hardware, firmware issues, or unstable physical connections.

Unstable InfiniBand links can interrupt RDMA communication, trigger NCCL communication failures, and affect distributed training workloads.

**Command**

```bash
# Check kernel logs for InfiniBand link state changes
journalctl -k --since "24 hours ago" --no-pager \
  | grep -iE 'mlx5.*(link down|link up|port.*down|port.*active)'

# Check RDMA and InfiniBand related kernel messages
dmesg -T | grep -iE 'mlx5_core|infiniband|link down|link up'

# Check current RDMA link state
rdma link show

# Check InfiniBand port counters
perfquery
```

**Expected Result**

- InfiniBand ports maintain stable `Active` and `LinkUp` states.
- No unexpected or recurring link down/up transitions are detected.
- No abnormal link recovery or physical link errors are observed.
- RDMA port counters do not show unexpected increases in link-related errors.

**Abnormal Symptoms**

- Repeated `Link down` and `Link up` messages appear in kernel logs.
- IB Port State frequently transitions between `Active` and `Down`.
- Physical State repeatedly changes between `LinkUp` and `Polling`.
- Link recovery and symbol error counters continue to increase.
- RDMA communication is interrupted or NCCL jobs experience communication failures.

**Troubleshooting**

- Identify the affected NIC, PCI BDF, and InfiniBand port from kernel logs.
- Inspect cable connections and optical transceiver health.
- Check switch-side port logs for corresponding link transitions.
- Inspect InfiniBand port counters using `perfquery` and `ibqueryerrors`.
- Verify NIC firmware and driver versions.
- Check NIC temperature and hardware health when supported.
- Compare the affected link with healthy ports on the same node or switch.
- Replace suspected faulty cables or transceivers after confirming the physical link issue.

---

### Check InfiniBand Network Interface Mapping

**Purpose**

Verify that RDMA devices and InfiniBand ports are correctly mapped to their expected Linux network interfaces.

Detect unexpected network interface name changes that may affect network configuration, monitoring, RDMA connectivity, and distributed training workloads.

**Command**

```bash
# Display RDMA device to network interface mapping
ibdev2netdev

# Display RDMA link information
rdma link show

# List all network interfaces
ip -br link

# Check RDMA device and port information
ibstat
```

**Expected Result**

- All expected RDMA devices are mapped to the correct network interfaces.
- Network interface names match the cluster configuration baseline.
- RDMA port-to-interface mapping remains consistent across reboots.
- Network interface states match the expected operational configuration.
- No unexpected device renaming or missing interfaces are detected.

Example:

```text
mlx5_0 port 1 ==> ib0 (Up)
mlx5_1 port 1 ==> ib1 (Up)
mlx5_2 port 1 ==> ib2 (Up)
mlx5_3 port 1 ==> ib3 (Up)
```

**Abnormal Symptoms**

- RDMA devices are not mapped to expected network interfaces.
- Network interface names unexpectedly change after reboot.
- Expected interfaces are missing or renamed.
- Multiple RDMA devices are mapped differently from the cluster baseline.
- Interface state is unexpectedly `Down`.

**Troubleshooting**

- Check current RDMA device mapping using `ibdev2netdev`.
- Verify network interface enumeration using `ip -br link`.
- Check PCIe BDF and RDMA device mapping using `mst status -v`.
- Inspect udev and systemd `.link` configuration for persistent naming rules.
- Check NetworkManager or systemd-networkd configuration.
- Compare interface names and RDMA mappings with healthy nodes.
- Update network configuration references if interface renaming is confirmed and intentional.

---

### Check RDMA NIC PCIe Link Speed and Width

**Purpose**

Detect PCIe link speed and width degradation on InfiniBand RDMA network adapters.

Verify that each NIC operates at its expected PCIe generation and lane width according to the hardware specification.

PCIe link degradation, such as Gen4 x16 downgraded to Gen4 x4, can significantly reduce host-to-NIC data transfer bandwidth and affect GPUDirect RDMA and NCCL communication performance.

**Command**

```bash
# Display MST device-to-PCI BDF and RDMA device mapping
mst status -v

# Check PCIe link speed and width for ConnectX-6 (CX6) NICs
mst status -v | grep -i x6 | grep mlx | awk '{print $3, $4}' | while read busid mlx_num; do
    printf "MLX Number: %s " "$mlx_num"
    lspci -vv -s "$busid" | grep -i 'lnksta:' | tr '\n' ' '
    echo
done

# MLX Number: mlx5_0           LnkSta: Speed 16GT/s (ok), Width x16 (ok)
# MLX Number: mlx5_1           LnkSta: Speed 16GT/s (ok), Width x4 (downgraded)
# MLX Number: mlx5_2           LnkSta: Speed 16GT/s (ok), Width x16 (ok)
# MLX Number: mlx5_3           LnkSta: Speed 16GT/s (ok), Width x16 (ok)
# MLX Number: mlx5_4           LnkSta: Speed 16GT/s (ok), Width x16 (ok)
# MLX Number: mlx5_5           LnkSta: Speed 16GT/s (ok), Width x16 (ok)
# MLX Number: mlx5_6           LnkSta: Speed 16GT/s (ok), Width x16 (ok)
# MLX Number: mlx5_7           LnkSta: Speed 16GT/s (ok), Width x16 (ok)

# ConnectX-7 (CX7) NICs
mst status -v | grep -i x7 | grep mlx | awk '{print $3, $4}' | while read busid mlx_num; do
    printf "MLX Number: %s " "$mlx_num"
    lspci -vv -s "$busid" | grep -i 'lnksta:' | tr '\n' ' '
    echo
done
```

Alternatively, inspect a specific NIC directly:

```bash
# Replace PCI_BDF with the actual NIC PCI address
lspci -vv -s <PCI_BDF> | grep -E 'LnkCap:|LnkSta:'
```

**Expected Result**

- NIC PCIe Link Speed matches the expected PCIe generation.
- NIC PCIe Link Width matches the expected lane width.
- No unexpected PCIe bandwidth degradation is detected.
- The negotiated Link Speed and Link Width match the expected hardware configuration.

Example of healthy PCIe link status:

```text
MLX Number: mlx5_0
LnkSta: Speed 16GT/s (ok), Width x16 (ok)

MLX Number: mlx5_1
LnkSta: Speed 16GT/s (ok), Width x16 (ok)
```

Example of abnormal PCIe link status:

```text
MLX Number: mlx5_0
LnkSta: Speed 16GT/s (ok), Width x16 (ok)

MLX Number: mlx5_1
LnkSta: Speed 16GT/s (ok), Width x4 (downgraded)
```

**Abnormal Symptoms**

- PCIe Link Speed is lower than the expected generation.
- PCIe Link Width is downgraded, such as x16 to x8 or x4.
- `LnkSta` reports `downgraded`.
- Different NICs with the same model and platform configuration report inconsistent PCIe link widths.
- RDMA bandwidth is significantly lower than expected despite a healthy IB port state.

**Troubleshooting**

- Identify the affected NIC and its PCI BDF using `mst status -v`.
- Compare current Link Status (`LnkSta`) against Link Capability (`LnkCap`).
- Inspect PCIe topology using `lspci -tv` and `lstopo`.
- Check PCIe AER logs for link errors.
- Inspect PCIe slots, risers, cables, and motherboard connections.
- Verify BIOS PCIe generation and lane configuration.
- Compare NIC PCIe link status with healthy nodes using the same hardware configuration.
- Check NIC firmware and PCIe switch status.
- Contact the hardware vendor if PCIe link degradation persists after physical and configuration checks.

**Note**

- InfiniBand port width (`4X`) and PCIe link width (`x16`, `x8`, `x4`) represent different interfaces and must be checked independently.
- PCIe Link Speed and Width should be compared against the NIC and platform specifications, not assumed to be identical across all ConnectX generations.
- A healthy InfiniBand Port State does not guarantee that the NIC PCIe link is operating at full bandwidth.
- PCIe link degradation can affect GPUDirect RDMA performance even when the InfiniBand port reports `Active`.