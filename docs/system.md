# System

Establish a clean and verifiable system baseline before installing optional software or applying customizations.

The objective is to understand the current installation, apply pending maintenance, verify system health, and identify hardware-specific requirements before making changes.

## Installation

Identify the Ubuntu release, kernel, architecture, and desktop session:

```bash id="n7v3kx"
cat /etc/os-release
uname -r
dpkg --print-architecture
echo "$XDG_SESSION_TYPE"
```

Ubuntu 26.04 Desktop uses **Wayland by default**. X11 applications remain supported through **XWayland**.

Record the output before making major system changes. This provides a useful reference when troubleshooting later.

## Hardware

Inspect the hardware relevant to the system configuration:

```bash id="m4q8zd"
lscpu
free -h
lsblk -o NAME,SIZE,FSTYPE,FSVER,MOUNTPOINTS
lspci
lsusb
```

Use the results to determine which hardware-specific configuration is actually required.

Do not install drivers, firmware, or hardware-specific software until the relevant hardware has been identified.

## Updates

Refresh the APT package index and apply available upgrades:

```bash id="k2w9pf"
sudo apt update
sudo apt upgrade
```

Reboot when required, particularly after kernel or other low-level system updates:

```bash id="v6r3hc"
sudo reboot
```

After reboot, verify the running kernel and failed systemd units:

```bash id="q8m5ty"
uname -r
systemctl --failed
```

> [!Warning]
> Do not interrupt an active `apt` or `dpkg` transaction. An interrupted package operation can leave the system in a partially configured state.

If package configuration was interrupted:

```bash id="c5n7wb"
sudo dpkg --configure -a
sudo apt --fix-broken install
```

Resolve package-management errors before continuing with unrelated system changes.

## Time

Check the current time, timezone, and synchronization status:

```bash id="x3m8vk"
timedatectl
```

List available timezones:

```bash id="d9q2sf"
timedatectl list-timezones
```

Set the timezone only when necessary:

```bash id="r6k4jp"
sudo timedatectl set-timezone Region/City
```

Correct system time is important for TLS certificate validation, authentication, logging, Git operations, scheduled tasks, and distributed systems.

## Power

Inspect the active power profile:

```bash id="w5c8qn"
powerprofilesctl get
```

List available profiles:

```bash id="a7m2yd"
powerprofilesctl list
```

Use `balanced` as the general-purpose baseline unless the workload requires another profile.

Avoid permanently forcing `performance` mode without measuring the effect on power consumption, thermals, noise, and actual workload performance.

## Storage

Inspect filesystem usage:

```bash id="p4j8xs"
lsblk -f
df -h
```

Identify large directories in the home directory:

```bash id="z6v3mc"
du -h --max-depth=1 "$HOME" | sort -h
```

For interactive disk usage analysis:

```bash id="e2k9rf"
ncdu "$HOME"
```

Storage usage should be monitored before disks approach capacity. Low free space can cause application failures, package-management problems, and degraded system behavior.

## System Health

Check for failed systemd units:

```bash id="u8n4kb"
systemctl --failed
```

Inspect errors reported during the current boot:

```bash id="f3m7qp"
journalctl -p err -b
```

Inspect the current kernel log:

```bash id="h5v2wd"
journalctl -k -b
```

A healthy baseline should be established before customization. If persistent hardware, service, or kernel errors are already present, diagnose them before adding unrelated software.

> [!Important]
> `systemctl --failed` showing no failed units does not prove that the system is completely error-free. It only indicates that systemd is not currently reporting failed units.

## Automatic Updates

Ubuntu uses `unattended-upgrades` to automate applicable package updates, particularly security updates.

Check its status:

```bash id="y7c3nx"
systemctl status unattended-upgrades
```

Inspect the automatic-update configuration:

```bash id="m9r5vk"
cat /etc/apt/apt.conf.d/20auto-upgrades
```

Do not disable automatic security updates simply because they are not visible during normal desktop use.

If a controlled update policy is required, adjust the policy deliberately rather than disabling the mechanism without replacement.

## Kernel

Check the currently running kernel:

```bash id="t4k8zs"
uname -r
```

List installed kernel packages:

```bash id="q6p3wd"
dpkg -l 'linux-image*' | grep '^ii'
```

Prefer Ubuntu's supported kernel packages and update path.

Do not replace the distribution kernel solely because another kernel has a higher version number. A newer kernel can introduce compatibility, driver, stability, or maintenance trade-offs.

Change the kernel when there is a concrete requirement, such as hardware support, a confirmed bug fix, or a workload-specific compatibility issue.

## Performance

Measure system behavior before attempting optimization.

For boot performance:

```bash id="v9m2kf"
systemd-analyze
```

Inspect the critical boot path:

```bash id="c4x7qn"
systemd-analyze critical-chain
```

`systemd-analyze blame` can provide additional information:

```bash id="j8w5rd"
systemd-analyze blame
```

Treat `blame` output carefully. A service taking a long time does not necessarily mean that it is responsible for slow boot, because service startup can occur concurrently.

The correct optimization workflow is:

```text
Measure
   ↓
Identify the bottleneck
   ↓
Change one variable
   ↓
Measure again
   ↓
Keep the change only if it improves the target
```

Avoid generic "Linux optimization" scripts and unexplained kernel parameters. System performance should be optimized for a measured workload rather than for arbitrary benchmark numbers.

## Baseline Checklist

Before installing optional software or heavily customizing the system, verify:

- Ubuntu release and architecture are known;
- hardware has been identified;
- pending system updates have been applied;
- the running kernel is known;
- time and timezone are correct;
- storage has sufficient free space;
- no unexplained systemd failures are present;
- automatic security updates are configured;
- the default power profile is appropriate;
- performance problems are measured rather than assumed.

This baseline makes subsequent customization easier to troubleshoot and revert.