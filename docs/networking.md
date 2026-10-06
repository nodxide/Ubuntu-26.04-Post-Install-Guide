# Networking

Network troubleshooting should follow a **layered model**. Verify the interface, link, routing, DNS, and application connectivity independently before changing configuration.

## Interface

Inspect network interfaces and addresses:

```bash id="4q1z4k"
ip -br addr
```

Check the interface state:

```bash id="zq0j5p"
ip link
```

A working interface should have an appropriate state and, for a configured network, an assigned IP address.

## Routes

Inspect the routing table:

```bash id="l2k8rs"
ip route
```

Check the default route specifically:

```bash id="x4z3pm"
ip route show default
```

A missing default route usually prevents access to networks outside the local subnet.

## Connectivity

Test basic IP connectivity without involving DNS:

```bash id="g7j2km"
ping -c 4 1.1.1.1
```

If this succeeds, the system can reach the destination by IP.

To test the local gateway:

```bash id="d1s8fv"
ip route show default
```

Then ping the reported gateway address:

```bash id="f8k3q2"
ping -c 4 <GATEWAY_IP>
```

This helps distinguish local network problems from upstream connectivity problems.

> [!Important]
> A failed `ping` does not always mean that the network is unavailable. Some hosts and networks intentionally block ICMP. Use multiple tests before concluding that connectivity is broken.

## DNS

Inspect the active resolver configuration:

```bash id="j7h2nv"
resolvectl status
```

Resolve a hostname:

```bash id="s6k4wp"
resolvectl query example.com
```

Test DNS independently from general connectivity:

```bash id="r3m9tx"
getent hosts example.com
```

Do not overwrite `/etc/resolv.conf` manually as a first troubleshooting step. Ubuntu normally manages DNS through its resolver and network configuration stack.

## HTTPS

Test application-level connectivity:

```bash id="v5c2qn"
curl -I https://example.com
```

This checks connectivity beyond ICMP and DNS, including TCP, TLS, and HTTP.

Interpret failures in layers:

```text
No interface
    → interface, link, or configuration problem

Interface but no IP
    → DHCP or address configuration problem

IP but no default route
    → routing problem

IP connectivity works but hostname fails
    → DNS problem

DNS works but HTTPS fails
    → TLS, proxy, firewall, routing, or application problem
```

## NetworkManager

Ubuntu Desktop normally uses NetworkManager for network configuration.

Check its status:

```bash id="h1q5sc"
systemctl status NetworkManager
```

Check general state:

```bash id="m6w8kr"
nmcli general status
```

List network devices:

```bash id="t9v4jd"
nmcli device status
```

List configured connections:

```bash id="b2n7xf"
nmcli connection show
```

For detailed information about a device:

```bash id="c8p1wy"
nmcli device show <DEVICE>
```

Prefer `nmcli` for NetworkManager-managed configuration rather than editing low-level configuration files without a specific reason.

## Wi-Fi

Inspect wireless devices:

```bash id="n4k7qs"
nmcli device
```

List visible networks:

```bash id="w2m5hr"
nmcli device wifi list
```

For Wi-Fi hardware and its kernel driver:

```bash id="e9r3kc"
lspci -nnk
```

For USB Wi-Fi adapters:

```bash id="p6v8dz"
lsusb
```

If the adapter is not detected at all, investigate the hardware and kernel driver before changing NetworkManager configuration.

## Firewall

Inspect listening network services:

```bash id="x7q2lm"
sudo ss -tulpn
```

Check the active UFW policy:

```bash id="k5r9vf"
sudo ufw status numbered
```

A service listening on a port does not automatically mean that the port is reachable from another machine. Check the listening address, firewall rules, routing, and network boundaries separately.

> [!Warning]
> Do not expose services to the network simply to test connectivity. Determine the required listening address and firewall policy first.

## Troubleshooting Workflow

Use the following order:

```text
Interface
    ↓
IP address
    ↓
Default route
    ↓
Gateway connectivity
    ↓
External IP connectivity
    ↓
DNS resolution
    ↓
HTTPS / application connectivity
```

Collect evidence at each stage before changing configuration.

Do not modify multiple unrelated network settings simultaneously. Change one variable, retest, and record the result.

> [!Important]
> Avoid treating DNS, routing, NetworkManager, and firewall configuration as interchangeable. A failure at one layer can produce symptoms that appear at another layer, so identify the failing layer first.