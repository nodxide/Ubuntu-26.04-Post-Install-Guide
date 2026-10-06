# Security

The goal is not maximum lockdown. The goal is a **defensible workstation with understood trust boundaries, least privilege, timely updates, and recoverable configuration**.

Security should be treated as a layered system rather than a single feature.

## Least Privilege

Use a normal user account for daily work and elevate privileges only when administrative access is required.

Verify the current user:

```bash id="3x7q9m"
whoami
```

Refresh the `sudo` authentication timestamp when administrative work is required:

```bash id="v5n2kc"
sudo -v
```

Avoid running desktop applications, development environments, or shells continuously as `root`.

> [!Important]
> `sudo` provides controlled privilege elevation; it does not make an application safe to run with elevated privileges. Only grant administrative access when it is actually required.

## AppArmor

Ubuntu uses **AppArmor** as a mandatory access-control mechanism. Profiles can restrict what applications are allowed to access even when those applications are running under an otherwise permitted Unix user.

Check AppArmor status:

```bash id="8q1m4f"
sudo aa-status
```

Inspect loaded profiles:

```bash id="r7c3wd"
sudo apparmor_status
```

When an application is blocked, do not disable AppArmor globally as a first response.

Instead:

1. identify the affected profile;
2. inspect the relevant denial;
3. determine whether the behavior is expected;
4. modify the application or profile only when necessary.

> [!Warning]
> Disabling AppArmor globally to resolve a single application problem removes protection from unrelated applications as well.

## Secure Boot

Check Secure Boot status:

```bash id="2x6k8p"
mokutil --sb-state
```

If enabled, keep Secure Boot enabled unless there is a specific technical requirement to change it.

Secure Boot provides a chain-of-trust mechanism for the boot process. Disabling it should therefore be treated as a deliberate security trade-off, not a generic troubleshooting step.

## Firewall

UFW provides a convenient interface for configuring the host firewall.

Inspect the current policy:

```bash id="9v4m2s"
sudo ufw status verbose
```

For a typical workstation, a reasonable baseline is:

```bash id="f6k8qd"
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
```

If SSH access is required, allow it **before** enabling the firewall:

```bash id="n3w7hj"
sudo ufw allow OpenSSH
```

Then verify the resulting rules:

```bash id="t2c9rx"
sudo ufw status numbered
```

> [!Warning]
> If you are connected to the machine remotely, verify the required management access before enabling or changing firewall rules. An incorrect rule can terminate your remote session.

A firewall does not make an insecure service secure.

Before exposing a service, inspect listening sockets:

```bash id="q8m5vd"
sudo ss -tulpn
```

Determine:

- which process owns the socket;
- which address it listens on;
- which port is exposed;
- whether external access is actually required.

Prefer binding services to `127.0.0.1` or another restricted interface when remote access is unnecessary.

## SSH

Use public-key authentication where possible.

Generate an Ed25519 key when creating a new key:

```bash id="w4p7sk"
ssh-keygen -t ed25519
```

Protect the SSH directory and private key:

```bash id="c6n2yf"
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

The private key must remain confidential. Only the public key should be copied to remote systems.

Verify the effective SSH configuration before troubleshooting:

```bash id="j9r3bx"
ssh -G <host>
```

> [!Important]
> Ed25519 is a strong modern default, but the appropriate SSH key type can depend on the target system and compatibility requirements.

## Updates

Security depends on keeping the operating system and installed software maintained.

Check the status of unattended upgrades:

```bash id="u5k8mz"
systemctl status unattended-upgrades
```

Inspect the package update configuration rather than disabling automatic security updates simply because they occur automatically.

Regularly review pending updates:

```bash id="p3d7qn"
sudo apt update
apt list --upgradable
```

Apply updates according to the system's maintenance requirements:

```bash id="e8w2vc"
sudo apt upgrade
```

> [!Important]
> Automatic updates reduce exposure to known vulnerabilities, but they do not replace regular maintenance, application updates, backups, or security monitoring.

## Secrets

Never commit sensitive credentials to a public repository.

This includes:

- private SSH keys;
- API tokens;
- passwords;
- cloud credentials;
- service-account credentials;
- database credentials;
- production `.env` files.

Use protected local configuration or an appropriate secret-management system.

For Git repositories, inspect the repository before committing sensitive configuration:

```bash id="a4m6ye"
git status
git diff --cached
```

A `.gitignore` entry can prevent accidental tracking, but it does **not** remove a secret that has already been committed.

> [!Warning]
> If a credential is accidentally committed, treat it as compromised. Remove it from the repository history if appropriate and, more importantly, revoke or rotate the credential.

## Security Model

Think about workstation security as multiple independent layers:

```text
Secure Boot
    ↓
Kernel and system updates
    ↓
AppArmor / application sandboxing
    ↓
Unix permissions and least privilege
    ↓
Host firewall
    ↓
Application configuration
    ↓
Secrets management
    ↓
Backups and recovery
```

Each layer addresses a different failure mode.

A firewall does not protect against a malicious application running locally. AppArmor does not replace correct file permissions. Secure Boot does not protect secrets after the system has booted. Backups do not prevent compromise, but they can significantly reduce the impact of data loss or ransomware.

## Security Baseline

A practical Ubuntu workstation baseline is:

- use a normal user account for daily work;
- retain Secure Boot where practical;
- keep AppArmor enabled;
- enable a restrictive host firewall policy when appropriate;
- expose only required network services;
- use SSH public-key authentication;
- keep security updates enabled;
- avoid unnecessary third-party repositories and software;
- keep credentials outside source repositories;
- maintain tested backups.

The objective is not to eliminate every possible attack surface. It is to **minimize unnecessary exposure, maintain clear trust boundaries, and make failures recoverable**.

## References

- [Ubuntu AppArmor Documentation](https://ubuntu.com/server/docs/how-to/security/apparmor/)
- [Ubuntu Firewall Documentation](https://documentation.ubuntu.com/security/security-features/network/firewall/)