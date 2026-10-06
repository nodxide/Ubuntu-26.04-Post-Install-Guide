# Troubleshooting

The most important troubleshooting rule is to **identify the failing layer before changing configuration**.

Start with evidence, form a hypothesis, change one variable, and verify the result. Avoid changing multiple unrelated settings at the same time.

## First Response

Collect a basic system snapshot:

```bash id="k3v8mp"
cat /etc/os-release
uname -r
systemctl --failed
journalctl -p err -b
df -h
free -h
```

Then classify the problem:

- boot or system startup;
- package management;
- graphics;
- networking;
- audio;
- storage;
- permissions;
- kernel or driver;
- service;
- application.

Do not begin by reinstalling packages or changing system configuration. First determine which layer is failing.

## Failed Services

List failed systemd units:

```bash id="m7q2xf"
systemctl --failed
```

Inspect a specific service:

```bash id="r5k9wd"
systemctl status <service>
```

Read its current-boot logs:

```bash id="c8v4ny"
journalctl -u <service> -b
```

For more detailed recent output:

```bash id="p2j6sk"
journalctl -u <service> -b --no-pager
```

A failed service is a symptom, not necessarily the root cause. Inspect its logs before restarting or reinstalling it.

## Package Management

If an APT or `dpkg` operation was interrupted:

```bash id="x4m8qz"
sudo dpkg --configure -a
sudo apt --fix-broken install
sudo apt update
```

Inspect package-management logs:

```bash id="w9f3kd"
less /var/log/dpkg.log
```

Check the package state when necessary:

```bash id="j6r2vc"
dpkg --audit
```

> [!Warning]
> Do not remove packages or repositories simply because an APT transaction failed. Determine which package or dependency caused the failure first.

## Boot Problems

From a working system, inspect the current boot:

```bash id="b7n4ys"
journalctl -b
```

Inspect kernel messages:

```bash id="q5k8mf"
journalctl -k -b
```

If the graphical environment does not start, switch to a virtual terminal:

```text
Ctrl + Alt + F3
```

Log in and collect system and service information from the TTY.

If the system does not boot normally, use the available recovery environment or previous kernel where appropriate before making permanent changes.

## Graphics

Identify the GPU and active kernel driver:

```bash id="v3m7px"
lspci -nnk
```

Check OpenGL:

```bash id="n8q4cw"
glxinfo -B
```

Check Vulkan:

```bash id="f6k2rz"
vulkaninfo --summary
```

For NVIDIA:

```bash id="h9w5sd"
nvidia-smi
```

Treat these as separate diagnostic layers. A working NVIDIA driver, OpenGL renderer, or Vulkan installation does not automatically prove that every application is using the expected GPU or graphics API.

## Networking

Inspect the interface and addresses:

```bash id="y4p8qm"
ip -br addr
```

Inspect routing:

```bash id="d6k3vx"
ip route
```

Inspect DNS:

```bash id="s8m2wf"
resolvectl status
resolvectl query example.com
```

Test HTTPS:

```bash id="a7q5nc"
curl -I https://example.com
```

Use the results to determine whether the failure is related to the interface, addressing, routing, DNS, TLS, firewall, or application.

Do not change DNS, NetworkManager, firewall rules, and kernel parameters simultaneously.

## DNS

If IP connectivity works but hostname resolution fails:

```bash id="p9w4jk"
ping -c 4 1.1.1.1
resolvectl query example.com
```

If the IP test succeeds while DNS resolution fails, focus on the resolver configuration rather than changing the entire network stack.

Inspect the active resolver configuration:

```bash id="e3r7mz"
resolvectl status
```

Do not overwrite `/etc/resolv.conf` as a first troubleshooting step.

## Storage

Check filesystem capacity:

```bash id="t8c2ny"
df -h
```

Inspect disks and filesystems:

```bash id="m5v9qx"
lsblk -f
```

Check available SMART devices:

```bash id="r2k6wd"
sudo smartctl --scan
```

For excessive home-directory usage:

```bash id="c7p4xs"
ncdu "$HOME"
```

Distinguish between a filesystem being full, a failing storage device, and an application generating excessive data. They require different solutions.

## Permissions

Inspect ownership and permissions:

```bash id="w3n8vf"
ls -la
```

Inspect the current user:

```bash id="j5q2mk"
id
```

Inspect group membership:

```bash id="s6r9pc"
groups
```

Do not use:

```bash
chmod -R 777 ...
```

This removes meaningful access restrictions and can create additional security problems.

Instead, identify:

- which user should own the file;
- which group requires access;
- what permissions are actually necessary;
- whether the application is running under the expected user.

## AppArmor

Check AppArmor status:

```bash id="f4k7zn"
sudo aa-status
```

If an application is denied access, identify the relevant profile and denial before modifying policy.

Do not disable AppArmor globally to resolve an application-specific problem.

## Docker

Check the Docker service:

```bash id="n8v3qy"
systemctl status docker
```

Inspect Docker's system state:

```bash id="k5m2wd"
docker info
```

List running containers:

```bash id="r7c9xf"
docker ps
```

Inspect published ports:

```bash id="v2j6mk"
sudo ss -tulpn
```

If a container cannot be reached, distinguish between container state, port publishing, host firewall rules, network binding, and the application inside the container.

## GNOME

If GNOME behaves unexpectedly, first determine whether the problem is caused by an extension or by the desktop environment itself.

Use the following isolation process:

1. Disable extensions.
2. Log out and back in.
3. Reproduce the problem.
4. Inspect relevant logs.
5. Re-enable extensions individually.

Avoid changing multiple GNOME settings before determining whether the issue persists with extensions disabled.

## Logs

Use `journalctl` to narrow logs by subsystem, service, or boot.

Current boot:

```bash id="d4x8sp"
journalctl -b
```

Errors from the current boot:

```bash id="q9m3vk"
journalctl -p err -b
```

Kernel messages:

```bash id="z6w2cn"
journalctl -k -b
```

Specific service:

```bash id="y5r7mf"
journalctl -u <service> -b
```

When investigating a problem, prefer the smallest relevant log scope instead of dumping the entire journal.

## Debugging Discipline

Use a controlled troubleshooting process:

```text
Observe
   ↓
Collect evidence
   ↓
Identify the failing layer
   ↓
Form a hypothesis
   ↓
Change one variable
   ↓
Retest
   ↓
Keep or revert the change
   ↓
Document the result
```

A useful diagnostic change should be **reversible, measurable, and specific to the hypothesis being tested**.

> [!Important]
> Avoid "configuration roulette": changing several unrelated settings and then being unable to determine which change fixed, introduced, or masked the problem.

The objective of troubleshooting is not merely to make the symptom disappear. It is to identify the underlying failure and leave the system in a configuration that remains understandable and maintainable.