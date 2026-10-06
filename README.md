# Ubuntu 26.04 LTS — Post-Install Guide

Production-oriented post-installation guidance for Ubuntu 26.04 LTS workstations.

This repository is not a collection of commands to blindly execute after installation. It provides a structured approach to configuring Ubuntu: what should be changed, why it matters, what should remain at the Ubuntu default, and how to verify and troubleshoot each change.

The goal is a stable, maintainable, and secure workstation rather than a heavily customized installation that becomes difficult to understand or maintain.

> **Core principle:** Verify → Understand → Change → Verify again.

## Scope

The guide covers:

- system updates and package management;
- security and hardening;
- hardware and graphics;
- networking and DNS;
- GNOME and desktop configuration;
- development tooling;
- containers and virtualization;
- gaming and AI/ML;
- backups and recovery;
- troubleshooting and maintenance.

Ubuntu 26.04-specific behavior is documented where it materially affects the procedure.

>[!Important]
>Do not execute every command in this repository blindly.
>
>Some sections are optional and hardware- or workload-dependent. In particular, do not install GPU compute stacks, virtualization components, container tooling, gaming software, or large application collections unless you actually need them.
>
>Do not disable security controls, remove Ubuntu components, add PPAs, or change kernel parameters simply because an optimization guide recommends doing so.

## Recommended workflow

1. Inspect the fresh installation.
2. Update Ubuntu.
3. Establish package-source policy.
4. Verify hardware and drivers.
5. Configure security and networking.
6. Install only the tooling required for your workload.
7. Configure backups.
8. Verify the final system.
9. Keep the configuration and recovery procedure documented.

## Documentation

| Document | Purpose |
| --- | --- |
| [System](docs/system.md) | Baseline inspection, updates, time, power, storage, maintenance |
| [Package Management](docs/package-management.md) | APT, Snap, Flatpak, repositories, PPAs |
| [Security](docs/security.md) | AppArmor, Secure Boot, UFW, SSH, least privilege |
| [Hardware](docs/hardware.md) | Hardware discovery, drivers, firmware, storage health |
| [Graphics](docs/graphics.md) | Mesa, NVIDIA, Vulkan, VA-API, Wayland/XWayland |
| [Networking](docs/networking.md) | NetworkManager, DNS, diagnostics, firewall boundaries |
| [Desktop](docs/desktop.md) | GNOME, extensions, fonts, audio, Bluetooth |
| [Development](docs/development.md) | Git, SSH, compilers, Python, editors, CLI tooling |
| [Containers](docs/containers.md) | Docker and container security |
| [Virtualization](docs/virtualization.md) | KVM, QEMU, libvirt, virt-manager |
| [Gaming](docs/gaming.md) | Steam, Vulkan, Proton and related tooling |
| [Backup & Recovery](docs/backup-and-recovery.md) | Backups, restore testing, recovery planning |
| [Troubleshooting](docs/troubleshooting.md) | Layered diagnosis and common recovery procedures |

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
