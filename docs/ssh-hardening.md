# SSH Hardening

## Overview

SSH access to the Kubernetes nodes is hardened using a VPN-only administration path, a jump host model, key-based authentication, restricted users, and Fail2ban protection.

The design follows a bastion-style access pattern where the Kubernetes control plane node acts as the SSH entry point for worker administration.

## Contents

- [SSH Access Model](#ssh-access-model)
- [Client ProxyJump Configuration](#client-proxyjump-configuration)
- [SSH Key Model](#ssh-key-model)
- [SSH Daemon Hardening](#ssh-daemon-hardening)
- [Fail2ban Protection](#fail2ban-protection)
- [Network Enforcement](#network-enforcement)
- [Validation](#validation)

## SSH Access Model

Administrative SSH access follows a VPN + jump host model.

```text
mgmt01
↓ WireGuard VPN
k8s-master - 10.10.10.10 - jump host
↓ SSH ProxyJump
worker nodes - 10.10.10.11-13
```

The access model is designed so that:

- `mgmt01` has direct SSH access only to `k8s-master`
- worker nodes are accessed through `k8s-master` using SSH ProxyJump
- direct SSH access from the VPN client to worker nodes is blocked
- worker administration follows a controlled bastion / jump host pattern

## Client ProxyJump Configuration

SSH aliases are configured on `mgmt01` using the local SSH client configuration.

Example `~/.ssh/config`:

```sshconfig
Host adam10
    HostName 10.10.10.10
    User adam

Host adam11
    HostName 10.10.10.11
    User adam
    ProxyJump adam10

Host adam12
    HostName 10.10.10.12
    User adam
    ProxyJump adam10

Host adam13
    HostName 10.10.10.13
    User adam
    ProxyJump adam10
```

This enables simplified administration commands:

```bash
ssh adam10
ssh adam11
ssh adam12
ssh adam13
```

Worker node SSH access is routed through the control plane node without exposing workers directly to the VPN client.

## SSH Key Model

SSH authentication uses keys rather than passwords.

The management workstation has key-based access to the control plane node.

The control plane node has key-based access to worker nodes for administrative operations.

Worker nodes also trust the management workstation public key, allowing ProxyJump connections from `mgmt01` through `k8s-master`.

The resulting model supports passwordless SSH administration while keeping direct network access to worker nodes restricted.

## SSH Daemon Hardening

SSH daemon hardening was applied to:

- `k8s-master`
- `worker1`
- `worker2`
- `worker3`

The following SSH daemon settings were configured:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
HostbasedAuthentication no
AllowUsers adam
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
```

These settings enforce:

- no direct root SSH login
- no password-based SSH login
- public key authentication only
- SSH access restricted to the `adam` user
- reduced authentication retry attempts
- shorter login grace period
- no X11 forwarding
- no host-based authentication

Configuration was validated before restarting SSH:

```bash
sshd -t
systemctl restart ssh
```

Changes were applied node by node to reduce the risk of administrative lockout.

## Fail2ban Protection

Fail2ban was installed on all Kubernetes nodes to protect SSH from repeated failed authentication attempts.

Installed on:

- `k8s-master`
- `worker1`
- `worker2`
- `worker3`

Example `/etc/fail2ban/jail.local` configuration:

```ini
[DEFAULT]
ignoreip = 127.0.0.1/8 10.20.20.2
bantime = 1h
findtime = 10m
maxretry = 3
backend = systemd

[sshd]
enabled = true
port = 22
logpath = %(sshd_log)s
```

The configuration provides:

- ban after 3 failed authentication attempts
- 10 minute detection window
- 1 hour ban time
- systemd journal backend
- VPN management client allow-listing

Fail2ban configuration was validated with:

```bash
fail2ban-client -t
systemctl restart fail2ban
fail2ban-client status
fail2ban-client status sshd
```

## Network Enforcement

The SSH hardening model is enforced both at the host level and through the existing network access model.

Final access behaviour:

| Flow | Result |
|------|------|
| `mgmt01 → k8s-master` | Allowed |
| `mgmt01 → worker nodes` | Blocked directly |
| `mgmt01 → k8s-master → worker nodes` | Allowed through ProxyJump |

This preserves least-privilege network access while maintaining practical administration workflows.

## Validation

The SSH access and hardening model was validated through direct and ProxyJump SSH testing.

Validated behaviour:

| Test | Result |
|------|------|
| SSH to `k8s-master` from `mgmt01` | Successful |
| Direct SSH to worker nodes from `mgmt01` | Blocked |
| SSH to worker nodes using ProxyJump | Successful |
| Password-based SSH login | Disabled |
| Root SSH login | Disabled |
| `sshd` configuration validation | Successful |
| Fail2ban configuration validation | Successful |
| Fail2ban SSH jail status | Active |

The final model provides VPN-only administrative SSH access, a controlled jump host workflow, key-based authentication, hardened SSH daemon settings, and Fail2ban protection.
