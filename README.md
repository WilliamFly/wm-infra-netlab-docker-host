# wm-infra-netlab-docker-host

Standalone, reusable Ansible role that installs Docker Engine (official
Docker apt repo — `docker-ce`, `docker-ce-cli`, `containerd.io`,
`docker-buildx-plugin`, `docker-compose-plugin`) on Ubuntu 24.04.

Used as a dependency by app repos that run a Docker-path deployment,
consumed via `ansible-galaxy install -r requirements.yml` — not copied.

## Usage

Add to the consuming repo's `ansible/requirements.yml`:

```yaml
roles:
  - src: https://github.com/WilliamFly/wm-infra-netlab-docker-host
    name: docker-host
```

Set which users get added to the `docker` group in that repo's
`group_vars`:

```yaml
docker_host_users:
  - netlab-admin
```

## Restricting Docker-published ports by source

Docker manages published container ports through its own `DOCKER-USER`/
`DOCKER` iptables chains, which sit in the `FORWARD` chain **ahead of and
independent of ufw**. ufw's own port rules (even ones with `from:` scoping)
do not apply to Docker-published ports at all — verified directly in this
project's lab (see ADR 0009 in `wm-infra-netlab`).

To restrict a published port by source subnet, set `docker_published_ports`
in the consuming repo's `group_vars`:

```yaml
docker_published_ports:
  - { port: "8081", proto: "tcp", from: "10.0.2.0/24" }
```

This role installs a small systemd service (`docker-user-firewall`) that
rebuilds the `DOCKER-USER` chain on every boot and every Ansible run —
`iptables-persistent`/`netfilter-persistent` is deliberately **not** used,
since it conflicts with `ufw` on Ubuntu (installing it silently removes
`ufw`).

## Requirements

- Ansible >= 2.15
- Ubuntu 24.04 (noble) — other versions untested
