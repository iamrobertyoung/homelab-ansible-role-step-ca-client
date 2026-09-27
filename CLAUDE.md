# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Ansible role (`step_ca_client`) for configuring step-ca client on Ubuntu/Debian systems. It installs step-cli, bootstraps the CA, generates service certificates, and sets up automatic renewal via a unified cert-manager script.

## Development Commands

### Testing
```bash
# Run full molecule test suite (starts an ephemeral step-ca container alongside the instance)
molecule test

# Run individual molecule stages
molecule converge  # Apply role
molecule verify    # Run verification
molecule destroy   # Cleanup
```

### Linting
```bash
yamllint .                    # Lint all YAML files
ansible-lint                  # Lint Ansible code
```

### Pre-commit Hooks
```bash
pre-commit install            # Install hooks
pre-commit run --all-files    # Run all hooks manually
```

### Tool Management
Uses `mise` for tool versions (Ansible 13, pipx 1.8). Run `mise install` to set up.

## Role Structure

- `tasks/main.yml` - Entry point, imports setup and client-cert tasks
- `tasks/setup.yml` - Adds step-ca repo, installs step-cli, bootstraps CA
- `tasks/client-cert.yml` - Deploys cert-manager script and systemd units
- `defaults/main.yml` - Default variables (step_ca_client_url, fingerprint, cert paths)
- `templates/cert-manager/` - Script and systemd templates for certificate management

## Required Variables

The role requires a running step-ca server. Key variables:
- `step_ca_client_url` - URL of the step-ca server
- `step_ca_client_fingerprint` - Fingerprint of the CA root certificate
- `fqdn` - Used for `step_ca_client_cert_name` if not overridden

## YAML Style

Line length max 120, truthy values must be "true"/"false"/"yes"/"no", 1 space minimum after comments.
