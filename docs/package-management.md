# Package Management

Ubuntu 26.04 uses **APT** as its primary system package manager. Ubuntu Desktop also integrates **Snap** into the application ecosystem, while **Flatpak** is an optional additional application format.

Use the package format that best fits the application and its integration requirements. Avoid installing the same application through multiple package formats without a specific reason.

## APT

Refresh the package index:

```bash
sudo apt update
```

List available upgrades:

```bash
apt list --upgradable
```

Upgrade installed packages:

```bash
sudo apt upgrade
```

Search for a package:

```bash
apt search <name>
```

Inspect package metadata:

```bash
apt show <package>
```

Install a package:

```bash
sudo apt install <package>
```

Remove a package while keeping its configuration files:

```bash
sudo apt remove <package>
```

Remove a package and its configuration files:

```bash
sudo apt purge <package>
```

Remove automatically installed dependencies that are no longer required:

```bash
sudo apt autoremove
```

> [!Important]
> `apt update` refreshes package metadata; it does not upgrade installed packages. Run `apt upgrade` separately when you are ready to apply available updates.

## Ubuntu Sources

Ubuntu 26.04 uses the **deb822** format for its default APT sources.

The main configuration directory is:

```text
/etc/apt/sources.list.d/
```

Ubuntu's default repository configuration is stored in:

```text
/etc/apt/sources.list.d/ubuntu.sources
```

Inspect the configured sources before modifying them:

```bash
cat /etc/apt/sources.list.d/ubuntu.sources
```

To inspect all configured repositories:

```bash
grep -R --no-filename -E '^(Types|URIs|Suites|Components|Signed-By):' \
  /etc/apt/sources.list.d/ 2>/dev/null
```

APT also supports additional `.sources` and legacy `.list` files in this directory.

> [!Warning]
> Do not modify Ubuntu's default repositories simply to follow an optimization guide. Changing mirrors, suites, components, or signing configuration can affect package availability and system updates.

## Third-Party Repositories

Treat every third-party APT repository as a **trust boundary**.

Before adding one, determine:

- whether Ubuntu already provides the package;
- whether the application is officially distributed as a Snap or Flatpak;
- whether the vendor maintains an official Ubuntu repository;
- whether the repository explicitly supports Ubuntu 26.04;
- which packages it provides or replaces;
- how repository signing keys are managed;
- how the repository is maintained and updated.

Prefer a vendor's **official repository** over an unofficial PPA or package mirror when a repository is genuinely required.

Third-party repositories can introduce package conflicts, dependency problems, security risks, and maintenance issues. Ubuntu recommends researching non-standard package sources carefully before adding them.

## PPAs

A PPA can be useful when a project does not provide a suitable package through Ubuntu or another official distribution channel.

Add a PPA only when there is a concrete reason:

```bash
sudo add-apt-repository ppa:<owner>/<name>
sudo apt update
```

Do not add large collections of PPAs simply to obtain newer versions of unrelated applications.

> [!Warning]
> A PPA changes the set of packages APT can install and may provide versions that interact with Ubuntu's own packages. Remove or disable obsolete PPAs rather than carrying them indefinitely between Ubuntu releases.

## Snap

Inspect installed snaps:

```bash
snap list
```

Refresh installed snaps:

```bash
sudo snap refresh
```

Inspect available information:

```bash
snap info <package>
```

Snaps are independently packaged applications managed by `snapd`. They can provide application versions and updates independently from the Ubuntu APT archive.

Ubuntu 26.04 also provides improved desktop integration for Snap applications, including better XDG portal support and permission management.

If an application is available as both a Snap and a deb package, choose deliberately based on application requirements, integration, update behavior, and maintenance preferences.

> [!Important]
> Do not remove Snap or `snapd` solely because an online post-install guide recommends doing so. Ubuntu Desktop uses Snap as part of its application ecosystem.

## Flatpak

Flatpak is an optional application distribution format that can complement APT and Snap.

Install Flatpak:

```bash
sudo apt install flatpak
```

Add Flathub if required:

```bash
flatpak remote-add --if-not-exists \
  https://flathub.org/repo/flathub.flatpakrepo
```

Search for an application:

```bash
flatpak search <application>
```

Install an application:

```bash
flatpak install flathub <application-id>
```

List installed Flatpaks:

```bash
flatpak list
```

Flatpak is primarily useful for desktop applications that benefit from application-level isolation and independently maintained versions.

## Package Provenance

Avoid installing the same application through APT, Snap, Flatpak, and a vendor-provided installer simultaneously unless there is a specific reason.

Multiple installations can create ambiguity around:

- executable paths;
- updates;
- permissions;
- configuration;
- desktop integration;
- file associations;
- disk usage.

Before installing an application, determine **which package format you actually want to maintain**.

A reasonable priority for a workstation is:

```text
Ubuntu package
    ↓
Official vendor package/repository
    ↓
Snap / Flatpak
    ↓
PPA or other third-party repository
    ↓
Manual installer
```

This is a guideline rather than a strict rule. The application's official distribution method and required version should determine the final choice.

## Interrupted APT Operations

If an APT or `dpkg` operation is interrupted, first complete pending package configuration:

```bash
sudo dpkg --configure -a
```

Then repair dependency problems:

```bash
sudo apt --fix-broken install
```

Refresh the package index:

```bash
sudo apt update
```

APT and `dpkg` activity is logged under:

```text
/var/log/dpkg.log
```

Do not immediately remove packages or repositories to recover from an interrupted transaction. Inspect the error first and repair the package state.

## Maintenance

A simple maintenance cycle is:

```bash
sudo apt update
apt list --upgradable
sudo apt upgrade
sudo apt autoremove
```

For Snap:

```bash
sudo snap refresh
```

Flatpak updates, when used:

```bash
flatpak update
```

Keep package sources minimal and intentional. A smaller, well-understood package configuration is easier to secure, troubleshoot, and maintain across Ubuntu releases.

## References

- [Ubuntu 26.04 Release Notes](https://documentation.ubuntu.com/release-notes/26.04/?utm_source=chatgpt.com)
- [Ubuntu Package Management Documentation](https://ubuntu.com/server/docs/how-to/software/package-management/?utm_source=chatgpt.com)
- [Ubuntu APT Source Configuration](https://ubuntu.com/project/docs/how-ubuntu-is-made/concepts/package-archive/?utm_source=chatgpt.com)
- [APT sources.list Manual](https://manpages.ubuntu.com/manpages/resolute/man5/sources.list.5.html?utm_source=chatgpt.com)