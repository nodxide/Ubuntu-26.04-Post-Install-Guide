# Backup and Recovery

A reliable workstation is not complete until its data and configuration can be recovered after hardware failure, accidental deletion, filesystem corruption, or a failed system change.

Backup planning should happen **before** extensive customization or development work begins.

The objective is not to back up everything. The objective is to ensure that important data can be restored within an acceptable amount of time and with an acceptable amount of data loss.

> **Principle:** A backup is only a backup if it can be restored.

---

## 1. Identify What Must Be Protected

Start by classifying data according to its importance and recoverability.

### Critical data

Data that would be difficult or impossible to recreate:

```text
~/Documents
~/Projects
~/Pictures
~/Videos
~/.ssh
~/.gnupg
```

Include other application-specific data if it cannot be recreated from another source.

### Configuration

Configuration that represents significant customization or development work:

```text
~/.gitconfig
~/.ssh/config
~/.config/
~/.local/bin/
~/bin/
shell configuration
editor configuration
terminal configuration
scripts
```

Not every file under `~/.config` needs to be backed up. Review application-specific data before including it.

### Reproducible data

Some data can be recreated from source repositories or package managers:

```text
Git repositories
Python virtual environments
Node.js dependencies
package caches
build directories
compiler caches
```

These generally do not need to be backed up if the original source and dependency definitions are available.

For example, prefer backing up:

```text
pyproject.toml
uv.lock
package.json
package-lock.json
Cargo.toml
Cargo.lock
```

rather than an entire generated dependency directory.

### Disposable data

Usually unnecessary to back up:

```text
~/.cache/
temporary files
browser caches
build artifacts
package caches
```

>[!Attention]
>Do not blindly back up the entire home directory.

This can significantly increase:

- backup size;
- backup duration;
- storage requirements;
- restore time;
- exposure of unnecessary data.

Classify the data first.

---

## 2. Separate Data Backups from Configuration Management

Git and backups solve different problems.

Git is appropriate for:

- source code;
- shell configuration;
- editor configuration;
- scripts;
- documentation;
- reproducible system configuration.

A backup system is appropriate for:

- personal files;
- photographs;
- application data;
- databases;
- SSH keys;
- non-versioned configuration;
- files that are not stored elsewhere.

For example:

```text
Git
 ├── dotfiles
 ├── scripts
 ├── development configuration
 └── documentation

Backup
 ├── personal documents
 ├── photographs
 ├── private keys
 ├── application data
 └── other irreplaceable files
```

>[!Attention]
>A Git repository is **not a backup**.
>
>A repository can be deleted, corrupted, compromised, or accidentally force-pushed. Important repositories should therefore also exist in an independent backup location.

---

## 3. Back Up Important Configuration

If your configuration is maintained as code, keep it in a dedicated repository.

Examples:

```text
~/.gitconfig
~/.ssh/config
~/.config/git/
~/.config/<application>/
~/.local/bin/
~/scripts/
```

A useful approach is to maintain a reproducible configuration repository such as:

```text
dotfiles/
├── shell/
├── git/
├── ssh/
├── editors/
├── terminal/
├── scripts/
└── README.md
```

This allows the configuration to be recreated on another system instead of relying exclusively on a binary backup.

### Attention: secrets

Never commit:

```text
private SSH keys
API tokens
passwords
cloud credentials
database credentials
.env files containing secrets
```

If sensitive configuration must be backed up, store it using an encrypted backup mechanism or an appropriate secret-management solution.

---

## 4. Choose a Backup Strategy

The appropriate backup method depends on what you need to recover.

Common approaches include:

- Déjà Dup — simple desktop backups;
- Borg — deduplicated, compressed, encrypted backups;
- Restic — encrypted, repository-based backups;
- rsync — flexible file synchronization;
- NAS — centralized local storage;
- external HDD/SSD — offline or periodically connected backups;
- encrypted cloud storage — off-site protection;
- filesystem snapshots — rapid local rollback.

These tools solve different problems.

For example:

```text
Filesystem snapshot
    ↓
Fast rollback
    ↓
Same system / same storage

Backup repository
    ↓
Data recovery
    ↓
Independent storage

Off-site backup
    ↓
Disaster recovery
    ↓
Different physical location
```

Do not choose a backup tool solely because it is popular. Choose it based on:

- recovery requirements;
- storage capacity;
- encryption requirements;
- retention requirements;
- automation;
- deduplication;
- off-site capability;
- restore speed;
- platform compatibility.

---

## 5. Follow the 3-2-1 Principle

A useful baseline is:

```text
3 copies of important data
        ↓
2 different types of storage
        ↓
1 copy stored off-site
```

For example:

```text
Primary workstation
        │
        ├── External backup drive
        │
        └── Encrypted off-site backup
```

The important property is **independence**.

Three copies on the same disk are not three meaningful copies.

---

## 6. Consider an Offline Backup

An external backup drive that is permanently connected to the workstation is vulnerable to some of the same incidents as the workstation itself.

For important data, consider periodically disconnecting one backup medium.

Example:

```text
Workstation
    ↓
Online backup
    ↓
External drive
    ↓
Disconnect
    ↓
Store separately
```

This provides additional protection against:

- ransomware;
- accidental deletion;
- filesystem corruption;
- electrical failure;
- some malware scenarios.

### Attention

An offline backup must still be maintained.

A drive that has not been updated for two years may technically be a backup while being practically useless.

---

## 7. Encrypt Backups Containing Sensitive Data

Backups frequently contain more information than the original workstation because they may preserve historical versions.

If a backup contains:

```text
SSH keys
documents
private photographs
credentials
financial records
personal data
```

encryption should be strongly considered.

The encryption key or password must itself be protected.

### Critical warning

Do not store the only copy of the backup encryption key inside the backup being encrypted.

Consider storing recovery credentials separately using a secure password manager or another protected mechanism.

---

## 8. Define Retention

A backup strategy should define how long historical versions are retained.

A simple example:

```text
Daily
 └── recent changes

Weekly
 └── recent historical state

Monthly
 └── long-term recovery points
```

The exact retention policy depends on the data.

For development projects, you may primarily need protection against accidental deletion.

For personal documents and photographs, long-term historical retention may be more important.

### Why retention matters

Without versioned backups, an accidentally deleted or corrupted file may be faithfully synchronized to every backup.

Version history allows you to recover an earlier state.

---

## 9. Back Up Databases Properly

Do not assume that copying a live database directory is a valid backup.

For databases, prefer application-aware backups such as:

```text
PostgreSQL
    ↓
pg_dump / pg_dumpall

MySQL / MariaDB
    ↓
mysqldump / mariadb-dump
```

Then back up the resulting dump using the normal backup system.

For larger deployments, use the database's documented backup and recovery mechanisms.

### Attention

A filesystem copy of a running database can produce an inconsistent backup unless the database and storage mechanism explicitly support that procedure.

---

## 10. Test Restores

A successful backup command does not prove that the backup is usable.

Periodically perform a restore test.

At minimum, restore:

- one representative document;
- one directory;
- one configuration file;
- one project;
- one historical version.

For critical systems, perform a complete recovery test in a separate environment.

### Example recovery test

```text
Backup repository
        ↓
Select historical snapshot
        ↓
Restore to temporary directory
        ↓
Verify files
        ↓
Verify permissions
        ↓
Verify application data
        ↓
Document result
```

Record:

- when the test was performed;
- what was restored;
- how long it took;
- whether anything failed.

> **A backup that has never been restored is an assumption, not a verified recovery mechanism.**

---

## 11. Protect File Permissions

When backing up Linux configuration, preserve ownership and permissions where the backup system supports it.

This is especially important for:

```text
~/.ssh/
~/.gnupg/
```

For example:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

After restoration, verify:

```bash
ls -la ~/.ssh
```

Incorrect permissions can prevent applications such as SSH from using restored credentials.

---

## 12. Recovery: Broken Package State

If an interrupted package operation leaves APT or `dpkg` in an inconsistent state:

```bash
sudo dpkg --configure -a
```

Then:

```bash
sudo apt --fix-broken install
```

Finally:

```bash
sudo apt update
sudo apt upgrade
```

Inspect failed services:

```bash
systemctl --failed
```

### Attention

Do not immediately reinstall Ubuntu because of a single package-management error.

First determine whether the package database can be repaired.

---

## 13. Recovery: Broken Graphical Session

If GNOME fails to start, switch to a TTY:

```text
Ctrl + Alt + F3
```

Log in and inspect:

```bash
systemctl --failed
```

```bash
journalctl -b -p err
```

```bash
journalctl -k -b
```

If the problem appeared immediately after installing a GNOME extension, disable the extension before making deeper system changes.

### Recovery principle

```text
Application
    ↓
GNOME extension
    ↓
Desktop configuration
    ↓
Display manager
    ↓
Graphics stack
    ↓
Kernel / driver
```

Start with the highest layer that changed recently.

---

## 14. Recovery: Broken Driver

If a graphics or hardware driver causes the graphical environment to fail:

1. switch to a TTY;
2. inspect the kernel and driver state;
3. identify the package that changed;
4. revert or repair the affected package;
5. reboot;
6. verify the graphics stack.

Useful commands:

```bash
lspci -nnk
```

```bash
uname -r
```

```bash
journalctl -k -b
```

For NVIDIA:

```bash
nvidia-smi
```

### Attention

Do not repeatedly install different driver versions until something works.

Record the current state first so that you know what changed.

---

## 15. Recovery: Configuration Error

If a configuration change breaks an application:

1. identify the configuration file;
2. restore the previous version;
3. reproduce the problem;
4. determine which setting caused it;
5. make the smallest necessary correction.

If configuration is stored in Git:

```bash
git log
git diff
git restore <file>
```

Use version control for configuration whenever practical.

---

## 16. Recovery: Accidental Deletion

If a file was accidentally deleted:

1. stop modifying the affected storage if data recovery may be necessary;
2. determine whether the file exists in a backup;
3. restore the required version;
4. verify its contents;
5. verify its permissions.

Prefer restoring from a known-good backup rather than relying on undelete utilities.

### Attention

Continued writes to the storage device can overwrite deleted data and reduce the possibility of forensic recovery.

---

## 17. Recovery: Disk Failure

A backup strategy should assume that the primary disk can fail completely.

Recovery should therefore be possible from:

```text
New / replacement disk
        ↓
Ubuntu installation
        ↓
System updates
        ↓
Configuration repository
        ↓
Backup repository
        ↓
Restore user data
        ↓
Restore applications
        ↓
Verify system
```

Do not design a backup system that requires the original disk to be operational.

---

## 18. Recovery: Complete Workstation Loss

For a serious workstation, document the information required to rebuild it.

Keep a recovery document containing:

```text
Ubuntu version
disk layout
important repositories
backup location
backup encryption/recovery information
hardware-specific requirements
required applications
development toolchains
configuration repository
network requirements
```

The goal is to make the following scenario realistic:

```text
New machine
    ↓
Install Ubuntu
    ↓
Restore configuration
    ↓
Restore data
    ↓
Reinstall required applications
    ↓
Resume work
```

This is much more valuable than merely having a large archive of files.

---

## 19. Snapshots Are Not Backups

Filesystem snapshots are useful for rapid rollback.

For example:

```text
Snapshot
    ↓
Fast recovery from a bad update
```

But a snapshot stored on the same physical disk does not protect against:

- disk failure;
- theft;
- filesystem destruction;
- hardware loss;
- some ransomware scenarios.

Therefore:

```text
Snapshot ≠ Backup
```

Snapshots and backups complement each other.

A robust workstation can use:

```text
Filesystem snapshots
        +
Local backup
        +
Off-site backup
```

---

### Final verification

Perform a restore test before assuming the backup system works.

```text
Backup created
     ↓
Backup verified
     ↓
Test restore
     ↓
Data verified
     ↓
Recovery procedure documented
```

Only after these steps should the workstation be considered **recovery-ready**.