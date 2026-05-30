# SSH Configuration Examples

This directory contains sanitised SSH configuration examples used for the Kubernetes node administration model.

The SSH access model follows a VPN + jump host pattern where `mgmt01` connects directly only to `k8s-master`, and worker nodes are administered through SSH ProxyJump.

## Files

| File | Purpose |
|------|------|
| [`ssh-config.example`](ssh-config.example) | Example SSH client aliases and ProxyJump configuration for `mgmt01` |
| [`sshd-hardening.conf.example`](sshd-hardening.conf.example) | Example SSH daemon hardening settings applied to Kubernetes nodes |
| [`fail2ban-jail.local.example`](fail2ban-jail.local.example) | Example Fail2ban SSH jail configuration |

## Access Model

| Flow | Result |
|------|------|
| `mgmt01 → k8s-master` | Allowed |
| `mgmt01 → worker nodes` | Blocked directly |
| `mgmt01 → k8s-master → worker nodes` | Allowed through ProxyJump |

## Notes

These examples are sanitised and do not include private keys.

The full SSH hardening design and validation notes are documented in [`docs/ssh-hardening.md`](../../docs/ssh-hardening.md).
