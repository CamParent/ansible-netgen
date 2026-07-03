# ansible-netgen
**Network Automation Toolkit — Cisco IOS & F5 BIG-IP**

A collection of Ansible automation projects targeting real-world network engineering workflows across multiple vendors. Covers both **configuration generation** (Cisco IOS) and **certificate lifecycle management** (F5 BIG-IP).

---

## Projects

### 1. Intended Configuration Generator — Cisco IOS
`playbooks/generate-intended.yml`

Renders validated, site-aware Cisco IOS configurations from structured variables and Jinja2 templates — **without connecting to live devices**. Demonstrates a safe, reviewable approach to network change management where intent is separated from execution.

**Key capabilities:**
- Input validation: interface naming, IPv4 format, duplicate detection, reserved range blocking
- Site-based baselines via `group_vars` (Columbus, Van Wert)
- Per-site AAA/TACACS+ definitions with Ansible Vault-encrypted secrets
- Optional DNS, NTP, and syslog configuration
- Idempotent, deterministic output — re-runs produce no changes unless inputs change

**Usage:**
```bash
ansible-playbook -i inventories/lab.ini playbooks/generate-intended.yml
```

---

### 2. F5 BIG-IP SSL Certificate Renewal
`playbooks/f5-cert-renew.yml` | `roles/f5_cert_renewal/`

Automates SSL certificate lifecycle management on F5 BIG-IP devices via the iControl REST API (`f5networks.f5_modules`). Built around a safety-first pipeline: **check → report → confirm → renew → verify**.

**Pipeline stages:**
1. **Check** — queries each BIG-IP for installed cert expiration dates via `bigip_device_info`
2. **Report** — flags certificates within the expiry threshold (default: 30 days)
3. **Dry-run gate** — stops here by default; nothing changes on the device unless `-e "confirm_renew=true"` is passed
4. **Archive** — renames the existing cert object with a timestamp before overwriting (manual rollback path)
5. **Renew** — uploads new key, cert, and chain; rebinds the `clientssl` profile
6. **Verify** — re-queries the device post-swap and asserts the new cert is present
7. **Audit log** — every check and renewal is appended to a local timestamped log

**Usage:**
```bash
# Reporting only — no changes made
ansible-playbook -i inventories/f5-hosts.yml playbooks/f5-cert-renew.yml --vault-id @prompt

# Execute renewal
ansible-playbook -i inventories/f5-hosts.yml playbooks/f5-cert-renew.yml --vault-id @prompt -e "confirm_renew=true"
```

**Requirements:**
```bash
ansible-galaxy collection install -r requirements.yml
```

---

## Repository Structure

```text
.
├── inventories/
│   ├── lab.ini                              # Cisco IOS lab inventory
│   └── f5-hosts.yml                         # F5 BIG-IP inventory
├── group_vars/
│   ├── cisco_ios.yml                        # Global IOS defaults
│   ├── columbus.yml                         # Site baseline — Columbus
│   ├── vanwert.yml                          # Site baseline — Van Wert
│   ├── cisco_ios_vault.yml                  # Vaulted TACACS secrets
│   └── f5_devices_vault_example.yml         # F5 credential vault reference
├── host_vars/
│   ├── core1.yml
│   └── branch1.yml
├── playbooks/
│   ├── generate-intended.yml                # Cisco IOS config generator
│   └── f5-cert-renew.yml                    # F5 SSL cert renewal entry point
├── roles/
│   └── f5_cert_renewal/
│       ├── defaults/main.yml                # Expiry threshold, cert list, paths
│       ├── tasks/
│       │   ├── main.yml                     # Pipeline orchestration
│       │   ├── check_expiry.yml             # Per-cert expiry lookup
│       │   └── renew_certificate.yml        # Archive → upload → rebind → verify
│       └── handlers/main.yml                # save sys config
├── templates/
│   └── ios_base.j2                          # Cisco IOS Jinja2 template
├── requirements.yml                         # Galaxy collections (f5networks.f5_modules)
├── ansible.cfg
└── .gitignore
```

---

## Design Principles

**Safety by default** — the F5 renewal playbook reports findings and stops unless explicitly confirmed. The Cisco IOS playbook never connects to live devices at all. Both approaches mirror how a production NetOps pipeline should behave: alert first, act on approval.

**Idempotency** — both projects produce the same output on repeated runs given the same inputs. F5 module calls are state-managed; re-running against an already-current cert is a no-op.

**Vault-first secrets** — no credentials in plaintext, no secrets committed to git. Both projects use Ansible Vault with separate vault files per device group.

**Audit trail** — the F5 project appends every check and renewal action to a local log with timestamp, host, cert name, and outcome.

---

## Security Notes
- TACACS+ shared keys stored in `group_vars/cisco_ios_vault.yml` (encrypted)
- F5 credentials stored in `group_vars/f5_devices/vault.yml` (encrypted, not committed)
- Generated IOS configs treated as build artifacts, not source-of-truth
- Private keys uploaded via `bigip_ssl_key` with `no_log: true`

---

## Future Enhancements
- GitHub Actions CI: ansible-lint + dry-run on PR
- Intended vs. running config diff for Cisco IOS
- Slack/Teams webhook notifications for cert expiry findings
- HashiCorp Vault PKI integration for cert/key retrieval
- Molecule test harness for F5 role

---

## Author

**Cameron Parent**  
Cloud & Infrastructure Security Engineer | CISSP | AZ-500 | CCNA  
GitHub: https://github.com/CamParent
