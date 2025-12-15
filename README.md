# ansible-netgen  
**Intended Network Configuration Generator (Cisco IOS)**

## Overview
`ansible-netgen` is an Ansible-based **intended configuration generator** for Cisco IOS devices.  
It renders validated, site-aware network configurations from structured variables and Jinja2 templates — **without connecting to live devices**.

This project demonstrates a **safe, scalable approach to network automation**, focusing on correctness, validation, and separation of intent from execution.

---

## Key Concepts Demonstrated
- Intended vs. applied configuration workflows
- Network automation with Ansible (no device access required)
- Jinja2 templating for IOS configurations
- Variable-driven design using `group_vars` and `host_vars`
- Strong input validation and guardrails
- Secure handling of sensitive data using Ansible Vault
- Idempotent automation runs

---

## Repository Structure
```text
.
├── inventories/
│   └── lab.ini                # Inventory definition
├── group_vars/
│   ├── cisco_ios.yml          # Global IOS defaults
│   ├── columbus.yml           # Site-specific baseline (Columbus)
│   ├── vanwert.yml            # Site-specific baseline (Van Wert)
│   └── cisco_ios_vault.yml    # Vaulted secrets (TACACS key)
├── host_vars/
│   ├── core1.yml              # Core device variables
│   └── branch1.yml            # Branch device variables
├── playbooks/
│   └── generate-intended.yml  # Main playbook
├── templates/
│   └── ios_base.j2            # IOS configuration template
├── intended/
│   └── *.cfg                  # Generated configs (gitignored in practice)
├── ansible.cfg
└── .gitignore
```

## What the Playbook Does

The `generate-intended.yml` playbook performs the following steps:

### 1. Validates Input Data
- Ensures required variables exist
- Enforces IOS-style interface naming conventions
- Validates IPv4 addresses using regex
- Rejects duplicate interface names and IP addresses
- Blocks reserved IPv4 ranges (loopback, multicast, etc.)
- Verifies TACACS+ definitions are structurally correct

### 2. Supports Enterprise-Ready Features
- Site-based baselines (e.g., **Columbus** vs **Van Wert**)
- Per-site AAA / TACACS+ server definitions
- Optional DNS, NTP, and SYSLOG configuration
- Secure TACACS key storage using **Ansible Vault**

### 3. Renders Intended Configurations
- Uses **Jinja2** templates
- Supports both dotted netmasks and CIDR notation
- Produces deterministic, idempotent output

---

## Example Usage

```bash
ansible-playbook -i inventories/lab.ini playbooks/generate-intended.yml
```

Re-running the playbook is **idempotent** — no changes are produced unless input variables or templates are modified.

---

## Security Considerations
- Sensitive data (e.g., TACACS shared keys) is stored in `group_vars/cisco_ios_vault.yml`
- Secrets are encrypted using **Ansible Vault**
- Vault files are never committed in plaintext
- Generated configurations are treated as **build artifacts**, not source-of-truth secrets

---

## Why Intended Configs?

Generating intended configurations allows teams to:

- Review and approve changes before deployment
- Integrate with CI/CD pipelines
- Perform configuration diffs against running devices
- Reduce risk in production environments

This mirrors how modern **NetOps, Cloud, and DevOps** teams manage infrastructure changes safely and predictably.

---

## Future Enhancements
Potential next steps for this project include:

- Intended vs. running configuration diffing
- GitHub Actions–based CI validation
- Multi-vendor device support
- Ansible linting and policy enforcement
- Automated compliance and security testing

---

## Author

**Cameron Parent**  
Network / Cloud Infrastructure Engineer  
GitHub: https://github.com/CamParent
