
# Virtualization

Ubuntu 26.04 LTS provides a KVM/QEMU/libvirt virtualization stack for running local virtual machines. KVM supplies hardware-assisted virtualization, QEMU provides the virtual machine backend, and libvirt provides the management API and tooling used by applications such as `virt-manager`.

Ubuntu 26.04 also introduces an optional Hardware Enablement (HWE) virtualization stack that receives newer virtualization components during the first two years of the LTS lifecycle.

## Hardware Virtualization

Before installing the virtualization stack, verify that the CPU exposes hardware virtualization.

```bash
lscpu | grep -i virtualization
```

For a more explicit check, install `cpu-checker` and run:

```bash
sudo apt install cpu-checker
kvm-ok
```

A supported system should report that KVM acceleration can be used. If virtualization is unavailable, check the firmware settings for Intel VT-x or AMD-V/SVM.

Check the loaded KVM modules:

```bash
lsmod | grep kvm
```

Typical systems will show either `kvm_intel` or `kvm_amd` together with the generic `kvm` module.

## KVM, QEMU and libvirt

For a standard Ubuntu workstation, install the base virtualization stack:

```bash
sudo apt update
sudo apt install \
    qemu-kvm \
    libvirt-daemon-system \
    libvirt-clients \
    virt-manager
```

Ubuntu's documentation uses `qemu-kvm` and `libvirt-daemon-system` as the core installation. `virt-manager` provides the graphical management interface and additional `virt-*` utilities.

Verify the libvirt service:

```bash
systemctl status libvirtd --no-pager
```

If necessary:

```bash
sudo systemctl enable --now libvirtd
```

Allow the current user to access the system-wide libvirt instance:

```bash
sudo adduser "$USER" libvirt
```

Log out and back in afterwards so the new group membership is applied. Ubuntu automatically adds members of the `sudo` group, but explicitly adding other users may be required.

Validate the host:

```bash
sudo virt-host-validate
```

This checks whether the host satisfies the requirements for virtualization and reports potential configuration problems.

> [!Important]
> The default Ubuntu configuration uses system-wide libvirt (`qemu:///system`). This provides shared VM management, system networking, automatic VM startup and stronger AppArmor integration than per-user session mode.

## virt-manager

Launch the graphical manager with:

```bash
virt-manager
```

`virt-manager` provides VM creation, lifecycle management, virtual hardware configuration and graphical consoles while using libvirt as the backend. It is primarily intended for workstations and test environments.

For command-line administration, use `virsh`:

```bash
virsh list --all
```

```bash
virsh net-list --all
```

```bash
virsh pool-list --all
```

These commands provide a useful overview of domains, networks and storage pools without relying on the GUI.

## Ubuntu 26.04 Virtualization HWE

Ubuntu 26.04 introduces an optional virtualization HWE stack containing:

- `qemu-hwe`
- `libvirt-hwe`
- `edk2-hwe`
- `seabios-hwe`

The stack is designed to provide newer virtualization components while retaining the Ubuntu LTS base. During the first two years of the LTS lifecycle, the HWE virtualization stack is updated approximately every six months to newer supported versions.

The base virtualization stack remains the default. HWE is an explicit opt-in choice rather than something that should be installed automatically on every workstation.

Inspect the current virtualization variant:

```bash
ubuntu_virt_helper --verbose
```

If the helper is not installed:

```bash
sudo apt install ubuntu-helper-virt-hwe
```

Use the helper when switching between the base and HWE variants:

```bash
sudo ubuntu_virt_helper
```

> [!Warning]
> Do not manually mix base and HWE virtualization packages. The two stacks are designed as mutually exclusive variants, and `ubuntu_virt_helper` performs the package transition while preserving the required dependency relationships and package installation state.

The HWE stack is most useful when you specifically need newer QEMU/libvirt/firmware functionality. For a normal workstation where the base stack already meets requirements, there is no need to switch simply because HWE exists.

## VM Networking

The default libvirt network normally provides NAT-based outbound connectivity through a virtual network interface.

Inspect available networks:

```bash
virsh net-list --all
```

Inspect the default network:

```bash
virsh net-info default
```

Inspect the host-side virtual bridge:

```bash
ip addr show virbr0
```

For most development and testing VMs, the default NAT network is sufficient.

Bridged networking is a separate configuration that allows guests to appear directly on the physical network. It should be configured deliberately rather than replacing the default network without a specific requirement.

## VM Storage

VM disks can consume significant amounts of storage, especially when using multiple operating systems or large QCOW2 images.

Check available disk space:

```bash
df -h
```

Inspect libvirt storage pools:

```bash
virsh pool-list --all
```

Inspect the default image directory:

```bash
sudo du -sh /var/lib/libvirt/images
```

For a more detailed filesystem analysis:

```bash
sudo du -xh /var/lib/libvirt/images | sort -h | tail
```

Keep VM images on storage with sufficient free space and avoid placing large images on a filesystem that is already close to capacity.

## VM Lifecycle

Common `virsh` operations include:

```bash
virsh list --all
```

```bash
virsh start <vm>
```

```bash
virsh shutdown <vm>
```

```bash
virsh destroy <vm>
```

```bash
virsh autostart <vm>
```

Use `shutdown` for a normal guest shutdown. `destroy` is equivalent to abruptly powering off the virtual machine and should be treated accordingly.

For persistent VM definitions, libvirt and `virt-manager` should generally be preferred over manually constructing long QEMU command lines.

## GPU Passthrough

GPU passthrough is an advanced KVM/libvirt configuration involving:

- IOMMU;
- VFIO;
- PCI device isolation;
- host and guest drivers;
- firmware configuration;
- GPU reset behavior;
- device ownership;
- guest firmware and machine configuration.

First inspect the PCI topology:

```bash
lspci -nnk
```

Inspect IOMMU groups:

```bash
find /sys/kernel/iommu_groups/ -type l | sort
```

GPU passthrough requires the target device to be suitable for assignment, normally including appropriate IOMMU isolation. BIOS/UEFI configuration may also be required.

> [!Warning]
> Do not blindly copy `intel_iommu=on`, `amd_iommu=on`, VFIO module configuration, GPU driver blacklists or other kernel parameters from generic passthrough guides. Determine the CPU platform, GPU topology, IOMMU groups and host driver requirements first.

GPU passthrough should be treated as a separate advanced configuration rather than part of the baseline Ubuntu virtualization setup.

## Baseline

For a typical Ubuntu workstation, the recommended baseline is:

```text
CPU virtualization enabled
        ↓
KVM
        ↓
QEMU
        ↓
libvirt
        ↓
virt-manager / virsh
        ↓
Virtual machines
```

Use the base Ubuntu virtualization stack unless newer virtualization components are specifically required. If the HWE variant is needed, transition to it as a complete stack using Ubuntu's virtualization helper rather than replacing individual packages.

Verify the final installation with:

```bash
kvm-ok
```

```bash
sudo virt-host-validate
```

```bash
virsh list --all
```

```bash
virsh net-list --all
```

```bash
virsh pool-list --all
```

This provides a minimal but complete validation of CPU virtualization, host configuration, VM management, networking and storage.