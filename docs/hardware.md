# Hardware

Hardware configuration should be **evidence-driven**. Identify the hardware and active drivers before installing or changing anything.

## Hardware Discovery

Inspect the CPU, PCI devices, USB devices, and general hardware inventory:

```bash
lscpu
lspci
lsusb
sudo lshw -short
```

For detailed PCI and driver information:

```bash
lspci -nnk
```

The `-k` option shows the kernel driver currently associated with PCI devices.

### Storage

List disks, partitions, filesystems, and mount points:

```bash
lsblk -f
```

For additional device information:

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,MODEL
```

### Memory

Check installed and available memory:

```bash
free -h
```

For detailed memory hardware information:

```bash
sudo dmidecode --type memory
```

## Drivers

Ubuntu generally provides hardware drivers through its repositories and automatically selects appropriate drivers where possible.

Inspect available driver recommendations:

```bash
ubuntu-drivers devices
```

List available driver packages:

```bash
ubuntu-drivers list
```

Install Ubuntu's recommended drivers when required:

```bash
sudo ubuntu-drivers install
```

Reboot after installing or changing low-level hardware drivers:

```bash
sudo reboot
```

> [!Important]
> Do not install drivers simply because they are available. First determine whether the hardware is already working correctly and whether a different driver provides a required feature or fixes a specific problem.

## Firmware

`fwupd` provides firmware updates for supported hardware through the Linux Vendor Firmware Service (LVFS).

Install it:

```bash
sudo apt install fwupd
```

List supported devices:

```bash
sudo fwupdmgr get-devices
```

Refresh firmware metadata:

```bash
sudo fwupdmgr refresh
```

Check for available updates:

```bash
sudo fwupdmgr get-updates
```

Install available updates:

```bash
sudo fwupdmgr update
```

> [!Warning]
> Firmware updates can affect device initialization and boot behavior. Keep a laptop connected to AC power and do not interrupt the update process.

Not every device supports firmware updates through `fwupd`. A device not appearing in `fwupdmgr get-devices` does not necessarily indicate a hardware problem.

## NVIDIA

For NVIDIA hardware, first identify the GPU and active driver:

```bash
lspci -nnk | grep -A3 -Ei 'vga|3d|display'
```

After installing the NVIDIA driver, verify the driver and GPU state:

```bash
nvidia-smi
```

Prefer Ubuntu-packaged NVIDIA drivers over vendor `.run` installers.

> [!Important]
> Avoid mixing Ubuntu's packaged NVIDIA driver with a manually installed `.run` driver. This can complicate package management, kernel updates, Secure Boot, and future driver changes.

## Storage Health

Install SMART utilities:

```bash
sudo apt install smartmontools
```

Detect available storage devices:

```bash
sudo smartctl --scan
```

Inspect a SATA/SAS drive:

```bash
sudo smartctl -a /dev/sdX
```

For NVMe devices, use the corresponding NVMe device path, for example:

```bash
sudo smartctl -a /dev/nvme0
```

SMART data can help identify reallocated sectors, media errors, wear, temperature problems, and other signs of device degradation.

> [!Important]
> SMART information is diagnostic evidence, not a guarantee that a drive will not fail. Maintain backups independently of reported drive health.

## USB and PCI Diagnostics

List USB devices:

```bash
lsusb
```

List PCI devices together with their kernel drivers:

```bash
lspci -nnk
```

For a specific PCI device:

```bash
lspci -nnk -s <PCI_ADDRESS>
```

Use these commands when a device is not detected, is using an unexpected driver, or requires identification before troubleshooting.

## Hardware Troubleshooting

When hardware does not work as expected, collect evidence before changing configuration:

```bash
lscpu
lspci -nnk
lsusb
lsblk -f
free -h
```

Then determine which layer is responsible:

```text
Hardware
    ↓
Firmware
    ↓
Kernel
    ↓
Kernel driver
    ↓
Userspace driver / service
    ↓
Application
```

Do not assume that installing a driver will fix every hardware problem. The issue may instead involve firmware, kernel support, device permissions, services, or application configuration.

> [!Warning]
> Do not install drivers for hardware you do not have. Do not disable Secure Boot merely to simplify an unsupported driver installation. Prefer Ubuntu-supported packages and configuration whenever they provide the required functionality.