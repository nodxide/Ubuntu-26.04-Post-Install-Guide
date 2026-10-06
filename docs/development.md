# Development

The system should provide a reliable base toolchain, while project-specific runtimes, dependencies, and configuration should remain isolated and reproducible.

## Build toolchain

Install the standard compiler, build, debugging, and system-tracing tools:

```bash
sudo apt update
sudo apt install \
  build-essential \
  pkg-config \
  cmake \
  ninja-build \
  gdb \
  strace \
  ltrace \
  valgrind
```

`build-essential` already provides GCC, G++, Make, and other essential build tools. Installing `make` separately is therefore unnecessary.

Verify the toolchain:

```bash
gcc --version
g++ --version
make --version
cmake --version
ninja --version
gdb --version
```

The installed system toolchain should be treated as the base environment. Individual projects may require different compiler or build-system versions.

## Git

Git should be configured globally for identity and general behavior:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global fetch.prune true
git config --global rerere.enabled true
```

Review the effective global configuration:

```bash
git config --global --list
```

Keep project-specific configuration inside the repository when it should not affect unrelated projects.

Useful examples include:

```bash
git config --local user.email "project@example.com"
git config --local core.editor "nvim"
```

Do not store credentials, access tokens, or other secrets in Git configuration files.

## SSH

SSH keys are commonly used for Git hosting, remote development, and server administration.

Generate an Ed25519 key:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Start the SSH agent and add the key:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Display the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

The public key can be added to the required Git hosting service or remote server.

Never expose or commit the private key:

```text
~/.ssh/id_ed25519
```

The public key is intended to be shared:

```text
~/.ssh/id_ed25519.pub
```

For persistent SSH configuration, use:

```text
~/.ssh/config
```

Set appropriate permissions on the SSH directory and private keys:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

## Python

Install the system Python development environment:

```bash
sudo apt install \
  python3 \
  python3-dev \
  python3-pip \
  python3-venv
```

Verify the installation:

```bash
python3 --version
python3 -m pip --version
```

Create an isolated project environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Verify that the environment is active:

```bash
which python
python --version
```

Deactivate it when finished:

```bash
deactivate
```

Do not install project dependencies into the system Python environment and do not use:

```bash
sudo pip install ...
```

For modern Python projects, prefer a project-oriented tool such as `uv` when appropriate. Keep project dependencies and their versions declared in the project rather than relying on globally installed packages.

## CLI tooling

Useful general-purpose development tools include:

```bash
sudo apt install \
  fzf \
  zoxide \
  ripgrep \
  fd-find \
  jq \
  btop \
  ncdu \
  tmux \
  neovim
```

Ubuntu may use distribution-specific executable names for some packages:

```text
fd   → fdfind
bat  → batcat
```

Verify the available commands rather than assuming upstream command names:

```bash
command -v fdfind
command -v batcat
```

These tools are optional and should be installed according to the actual development workflow.

## Editors

Choose a primary editor and configure it deliberately.

A terminal-oriented workflow can use Neovim, while larger projects may benefit from a full IDE or graphical editor.

Avoid installing multiple large IDEs unless they provide clearly different capabilities or workflows.

Keep editor configuration separate from project source code where appropriate:

```text
~/.config/<editor>/
```

Project-specific editor configuration should remain in the repository only when it benefits the entire development team.

## Project isolation

Do not install language runtimes, libraries, or project dependencies globally merely to satisfy a single project.

Prefer project-specific environments and dependency declarations:

```text
Project
├── runtime/version
├── dependencies
├── lockfile
├── build configuration
└── development tools
```

This prevents unrelated projects from depending on the same mutable global environment.

For Python, this typically means a virtual environment or `uv`-managed project. Other ecosystems should use their respective environment and dependency-management mechanisms.

## Development workflow

A general development workflow should remain reproducible:

```text
Environment
    ↓
Repository
    ↓
Runtime
    ↓
Dependencies
    ↓
Build
    ↓
Test
    ↓
Debug / Inspect
    ↓
Commit
```

Prefer explicit project configuration over undocumented workstation state.

A project should be understandable and reproducible on another development machine without depending on manually installed global packages.

## Maintenance

Keep the development environment under control:

- remove unused runtimes and tools;
- update development dependencies deliberately;
- avoid unnecessary global packages;
- keep project lockfiles current;
- periodically review SSH keys and configuration;
- keep compilers, debuggers, and build tools updated;
- remove obsolete virtual environments and caches.

Do not optimize for the smallest possible workstation installation. Keep commonly used development infrastructure available, but avoid turning the system-wide environment into a dependency store for individual projects.

## Development security checklist

Before using the workstation for development:

- configure Git identity explicitly;
- use SSH keys instead of passwords where appropriate;
- protect private SSH keys;
- avoid `sudo pip install`;
- isolate project dependencies;
- keep secrets outside source control;
- use `.env.example` instead of committing real `.env` files;
- keep runtimes and dependencies project-specific;
- verify third-party dependencies before installing them;
- keep important configuration reproducible;
- avoid relying on undocumented global system state.

> [!Important]
> The system development environment should provide the tools required to build, debug, and maintain software. Individual projects should own their runtime versions, dependencies, and reproducible configuration. This separation prevents one project from destabilizing another and makes the workstation easier to maintain.