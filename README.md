# homelab-ansible-role-step-ca-client

[![Lint](https://github.com/iamrobertyoung/homelab-ansible-role-step-ca-client/actions/workflows/lint.yml/badge.svg)](https://github.com/iamrobertyoung/homelab-ansible-role-step-ca-client/actions/workflows/lint.yml)
[![Release](https://github.com/iamrobertyoung/homelab-ansible-role-step-ca-client/actions/workflows/release.yml/badge.svg)](https://github.com/iamrobertyoung/homelab-ansible-role-step-ca-client/actions/workflows/release.yml)
[![SLSA 3](https://slsa.dev/images/gh-badge-level3.svg)](https://slsa.dev)
[![OpenSSF Scorecard](https://api.scorecard.dev/projects/github.com/iamrobertyoung/homelab-ansible-role-step-ca-client/badge)](https://scorecard.dev/viewer/?uri=github.com/iamrobertyoung/homelab-ansible-role-step-ca-client)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Ansible role for configuring step-ca client on Debian/Ubuntu systems. Installs step-cli, bootstraps the CA, generates service certificates, and sets up automatic renewal.

## Requirements

- Ansible >= 2.15
- Target: Ubuntu (focal, jammy, noble) or Debian (bullseye, bookworm)
- A running step-ca server

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `step_ca_client_url` | `https://step-ca.local.iamrobertyoung.co.uk` | URL of the step-ca server |
| `step_ca_client_fingerprint` | *(required)* | Fingerprint of the CA root certificate |
| `step_ca_client_cert_name` | `{{ fqdn }}` | Certificate common name |
| `step_ca_client_cert_file` | `/etc/ssl/certs/{{ step_ca_client_cert_name }}.crt` | Path to store the certificate |
| `step_ca_client_key_file` | `/etc/ssl/private/{{ step_ca_client_cert_name }}.pem` | Path to store the private key |
| `step_ca_client_provisioner_password` | `password` | Provisioner password for certificate requests |
| `step_ca_client_provisioner_password_file` | `/root/.step/config/ansible_provisioner_password.txt` | Path to store provisioner password |
| `step_ca_client_cert_not_after` | `24h` | Certificate validity duration (e.g., `24h`, `720h` for 30 days) |
| `step_ca_client_post_renew_commands` | `[]` | List of commands to run after certificate renewal |

## Usage

### Install via requirements.yml

```yaml
- src: git@github.com:iamrobertyoung/homelab-ansible-role-step-ca-client.git
  scm: git
  version: main
  name: step_ca_client
```

```bash
ansible-galaxy install -r requirements.yml
```

### Example Playbook

```yaml
- hosts: servers
  become: true
  roles:
    - role: step_ca_client
```

### Override defaults

```yaml
- hosts: servers
  become: true
  roles:
    - role: step_ca_client
      vars:
        step_ca_client_url: https://ca.example.com
        step_ca_client_fingerprint: "your-ca-fingerprint-here"
        step_ca_client_cert_name: "{{ inventory_hostname }}"
```

### Post-renewal commands

Run custom commands after certificate renewal (e.g., copy certs, change permissions, reload services):

```yaml
- hosts: servers
  become: true
  roles:
    - role: step_ca_client
      vars:
        step_ca_client_post_renew_commands:
          - "cp {{ step_ca_client_cert_file }} /etc/nginx/ssl/"
          - "cp {{ step_ca_client_key_file }} /etc/nginx/ssl/"
          - "systemctl reload nginx"
```

## What Gets Installed

- Smallstep APT repository and GPG key
- step-cli package
- CA bootstrap configuration (`/root/.step/`)
- Service certificate and private key
- Systemd timer for automatic certificate renewal

## Supply Chain Security

This project implements [SLSA](https://slsa.dev/) Level 3 provenance for release artifacts.

- Provenance attestations are submitted to [GitHub Attestations](https://github.com/iamrobertyoung/homelab-ansible-role-step-ca-client/attestations)
- Release artifacts include `.intoto.jsonl` provenance files
- Security posture tracked via [OpenSSF Scorecard](https://scorecard.dev/viewer/?uri=github.com/iamrobertyoung/homelab-ansible-role-step-ca-client)

### Verifying Release Provenance

```bash
# Using GitHub CLI (recommended)
gh attestation verify step_ca_client-<VERSION>.tar.gz \
  --repo iamrobertyoung/homelab-ansible-role-step-ca-client

# Or using slsa-verifier
VERSION="v1.0.0"  # Replace with desired version
slsa-verifier verify-artifact step_ca_client-${VERSION}.tar.gz \
  --provenance-path step_ca_client-${VERSION}.tar.gz.intoto.jsonl \
  --source-uri github.com/iamrobertyoung/homelab-ansible-role-step-ca-client \
  --source-tag "${VERSION}"
```

## License

MIT
