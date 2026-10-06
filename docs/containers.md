# Containers

Containers provide isolated application environments and are useful for local development, CI reproduction, integration testing, self-hosted services, and reproducible application stacks.

On a workstation, choose the container runtime and installation source deliberately. Avoid mixing instructions from different Docker installation methods, because package ownership, service configuration, upgrades, and security behavior can differ.

## Docker

Ubuntu provides Docker through the `docker.io` package. For many development and local-service workloads, the Ubuntu package is sufficient and has the advantage of being integrated with Ubuntu's package management.

Install Docker and the Compose v2 plugin:

```bash
sudo apt update
sudo apt install docker.io docker-compose-v2
```

Enable the Docker daemon:

```bash
sudo systemctl enable --now docker
```

Verify the installation:

```bash
docker version
docker compose version
docker run --rm hello-world
```

Check the service state if required:

```bash
systemctl status docker
```

### Docker installation source

There are two common approaches:

- Ubuntu's `docker.io` package;
- Docker's official upstream repository.

For a general workstation, prefer one source and remain consistent with it. Do not install Docker from the Ubuntu repositories and then add Docker's upstream repository simply to obtain another package version unless there is a specific requirement.

>[!Warning]
>Do not follow installation instructions that mix `docker.io`, Docker's upstream packages, Snap packages, and third-party repositories. This can create conflicting packages and make future maintenance unnecessarily difficult.

## Docker permissions

By default, Docker commands require access to the Docker daemon's privileged socket.

A common convenience is to add the current user to the `docker` group:

```bash
sudo usermod -aG docker "$USER"
```

This removes the need to use `sudo` for most Docker commands, but it also changes the security model.

Membership in the `docker` group effectively provides root-equivalent control over the host because Docker can create privileged containers, mount host filesystems, manipulate namespaces, and otherwise interact with the host at a high privilege level.

After changing group membership, start a new login session before relying on the new membership.

>[!Important]
>Do not treat the `docker` group as an ordinary application-access group. On systems where strong privilege separation matters, consider whether granting Docker daemon access to an unprivileged account is appropriate.
>
>For higher-isolation workloads, evaluate rootless Docker or another rootless container runtime instead.

## Compose

Docker Compose v2 is integrated with the Docker CLI as the `docker compose` subcommand.

Verify it with:

```bash
docker compose version
```

Use:

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs
```

Prefer `docker compose` over the obsolete standalone `docker-compose` command.

For projects, keep Compose definitions in version control where appropriate:

```text
compose.yaml
compose.override.yaml
.env.example
```

Never commit credentials, API tokens, private keys, or other secrets to a Compose file or `.env` file.

Use `.env.example` to document the required variables without exposing their actual values.

## Container networking

Published container ports can make services accessible beyond the container itself.

Inspect running containers:

```bash
docker ps
```

Inspect published ports:

```bash
docker ps --format 'table {{.Names}}\t{{.Ports}}'
```

Inspect listening sockets on the host:

```bash
sudo ss -tulpn
```

When exposing a service, prefer binding it only to the interfaces that actually require access.

For example:

```yaml
ports:
  - "127.0.0.1:8080:8080"
```

This exposes the service only through the host's loopback interface.

By contrast:

```yaml
ports:
  - "8080:8080"
```

can expose the published port on the host's network interfaces, depending on the Docker networking configuration.

### Firewall considerations

Do not assume that a host firewall policy alone completely describes the exposure of Docker-published ports.

Docker manages its own networking and firewall rules. A port published by a container therefore requires explicit verification of:

- the address to which the port is bound;
- Docker's networking configuration;
- host firewall behavior;
- whether the service itself performs authentication and authorization.

After deploying a network-facing container, verify the resulting exposure rather than relying solely on the Compose configuration.

## Storage

Docker images, containers, volumes, and build caches can consume substantial disk space.

Inspect Docker's storage usage:

```bash
docker system df
```

List volumes:

```bash
docker volume ls
```

List images:

```bash
docker image ls
```

List containers, including stopped containers:

```bash
docker ps -a
```

Avoid routinely running destructive cleanup commands.

For example:

```bash
docker system prune
```

removes unused Docker resources according to Docker's pruning rules. More aggressive variants can also remove unused images and volumes.

>[!Important]
>Do not blindly prune volumes on a workstation running stateful services. A Docker volume may contain the only copy of an application's persistent data.
>
>Treat container volumes as application data and include important volumes in your backup strategy.

## Persistent data

Containers should generally be treated as disposable. Persistent application state should reside in explicitly managed volumes or bind mounts.

Typical examples include:

```text
PostgreSQL data
MariaDB data
Redis persistence
application uploads
configuration generated at runtime
```

Before removing a container or volume, determine whether it contains data that must be preserved.

For databases, prefer application-aware backup procedures rather than relying solely on filesystem-level copies of a live database volume.

## Rootless containers

Rootless container workflows run container components without requiring a root-owned Docker daemon for normal operation.

They can reduce the impact of a compromised container workload and are worth considering when the workload and networking requirements support them.

Rootless operation is not a universal replacement for privileged Docker. Some workloads require capabilities, networking features, device access, or other functionality that may require additional configuration.

Choose the security model deliberately rather than combining unrelated instructions from rootful and rootless Docker guides.

## Container lifecycle

A useful development workflow is:

```text
Build
  ↓
Run
  ↓
Test
  ↓
Inspect logs
  ↓
Verify networking
  ↓
Persist required data
  ↓
Remove disposable resources
```

Useful commands include:

```bash
docker ps
docker images
docker volume ls
docker network ls
docker logs <container>
docker inspect <container>
```

For Compose-based projects:

```bash
docker compose ps
docker compose logs
docker compose config
```

`docker compose config` is particularly useful for validating the effective Compose configuration before starting a stack.

## Maintenance

Keep the following under control:

- unused images;
- stopped containers;
- obsolete build caches;
- unused networks;
- abandoned volumes;
- oversized container logs.

Do not optimize for minimal disk usage at the expense of recoverability. Clean up disposable resources, but preserve application data and anything required to reproduce the development environment.

## Container security checklist

Before using containers on a workstation:

- use one deliberate Docker installation source;
- keep Docker and Compose updated;
- understand the privileges of the Docker daemon;
- treat `docker` group membership as privileged access;
- consider rootless containers when appropriate;
- publish only the ports that are actually required;
- verify host-side network exposure;
- never store secrets in images or source-controlled Compose files;
- treat container volumes as persistent application data;
- back up important volumes and databases;
- avoid indiscriminate `prune` operations;
- inspect container images and their provenance;
- do not assume that container isolation is equivalent to a virtual machine.

>[!Warning]
>Containers provide process and filesystem isolation, not the same security boundary as a dedicated virtual machine. Do not use containers as a substitute for VM-level isolation when the threat model requires a stronger boundary.