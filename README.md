# ansible-aap-secure

Scan-and-remediate toolkit for AAP 2.7 security hardening based on the Red Hat AAP Secure guide. Designed for customer engagements - replaces a 3-4 day manual security assessment with automated scanning and remediation.

## Quick Start

```bash
# Install dependencies
ansible-galaxy collection install -r requirements.yml

# Run security scan (read-only)
ansible-playbook scan.yml -i inventory/sample_inventory.yml

# Run with custom inventory
ansible-playbook scan.yml -i /path/to/customer/inventory.yml

# Remediate (apply fixes)
ansible-playbook scan.yml -e aap_secure_mode=remediate

# Scan API-level only
ansible-playbook scan.yml -e aap_secure_scope=api

# Scan hosts only
ansible-playbook scan.yml -e aap_secure_scope=hosts

# With ansible-navigator
ansible-navigator run scan.yml -i inventory/sample_inventory.yml -m stdout
```

## Modes

| Mode | Description |
|---|---|
| `scan` | Read-only checks, produces compliance report (default) |
| `remediate` | Applies security fixes based on baseline + customer overrides |

## Scope

| Scope | Targets |
|---|---|
| `all` | API + hosts + managed nodes (default) |
| `api` | Gateway settings, authentication, RBAC, credentials, logging |
| `hosts` | Host hardening, TLS/certificates, managed nodes |

## Security Domains

### API-Level (via AAP REST API)

| Domain | Role | Key Checks |
|---|---|---|
| Gateway Settings | `gateway_security` | SESSION_COOKIE_SECURE, SESSIONS_PER_USER, CSRF, OAuth2 |
| Authentication | `authentication` | Authenticator type, LDAP/SAML/OIDC config, local auth |
| RBAC | `rbac` | Superuser count, org/team/role assignments, activity stream |
| Credentials | `credentials` | External vault integration, no_log, plaintext detection |
| Logging | `logging` | Log aggregator, activity stream, external logging |

### Host-Level (via SSH)

| Domain | Role | Key Checks |
|---|---|---|
| Host Hardening | `host_hardening` | fapolicyd, user namespaces, noexec, log permissions, session timeout |
| TLS/Certificates | `tls_certificates` | HTTPS/HSTS, PostgreSQL SSL, certificate expiry |

### Managed Nodes (via SSH)

| Domain | Role | Key Checks |
|---|---|---|
| Managed Nodes | `managed_nodes` | Service account, sudoers, SSH key auth, pam_access |

## Configuration

### Variables

| File | Purpose |
|---|---|
| `vars/security_baseline.yml` | Default hardened values from AAP Secure 2.7 guide |
| `vars/customer_overrides.yml` | Customer-specific exceptions and values |
| `vars/scan_policy.yml` | Mode, scope, and domain toggles |

### Domain Toggles

```yaml
aap_secure_check_gateway: true
aap_secure_check_auth: true
aap_secure_check_rbac: true
aap_secure_check_credentials: true
aap_secure_check_logging: true
aap_secure_check_host_hardening: true
aap_secure_check_tls: true
aap_secure_check_managed_nodes: true
```

## Report Output

Reports are generated in `reports/` directory (gitignored). Each report shows:

- Per-check status: PASS / FAIL / WARNING / SKIPPED
- Current value vs recommended value
- Remediation guidance
- Reference to AAP Secure guide section
- Overall compliance percentage

## Project Structure

```
ansible-aap-secure/
├── scan.yml                        # Main playbook (scan + remediate)
├── ansible.cfg
├── ansible-navigator.yml
├── requirements.yml
├── roles/
│   ├── gateway_security/           # Gateway settings checks
│   ├── authentication/             # Authenticator config checks
│   ├── rbac/                       # RBAC and superuser audit
│   ├── credentials/                # Credential hygiene checks
│   ├── logging/                    # Log aggregator checks
│   ├── host_hardening/             # Host-level security controls
│   ├── tls_certificates/           # TLS/HSTS and cert validation
│   ├── managed_nodes/              # Managed node hardening
│   └── report/                     # Compliance report generation
├── vars/
│   ├── security_baseline.yml       # Hardened baseline values
│   ├── customer_overrides.yml      # Per-customer exceptions
│   └── scan_policy.yml             # Scan configuration
├── templates/
│   └── compliance_report.html.j2   # HTML report template
├── inventory/
│   └── sample_inventory.yml        # Example inventory
└── reports/                        # Generated reports (gitignored)
```

## Prerequisites

- Ansible Automation Platform 2.7+
- Collections: `ansible.platform`, `infra.aap_configuration`
- For API checks: AAP admin credentials (read access for scan, write for remediate)
- For host checks: SSH access with become privileges

## Reference

- [Red Hat AAP Secure 2.7 Guide](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.7#Secure)
