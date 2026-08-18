# ansible-aap-secure

Scan-and-remediate toolkit for AAP 2.7 security hardening based on the Red Hat AAP Secure guide. Designed for customer engagements - replaces a 3-4 day manual security assessment with automated scanning and remediation.

## Quick Start

```bash
# Install dependencies
ansible-galaxy collection install -r requirements.yml

# Run security scan (read-only)
ansible-playbook aap_secure.yml -i inventory/sample_inventory.yml

# Run with custom inventory
ansible-playbook aap_secure.yml -i /path/to/customer/inventory.yml

# Remediate (apply fixes)
ansible-playbook aap_secure.yml -e aap_secure_mode=remediate

# Scan API-level only
ansible-playbook aap_secure.yml -e aap_secure_scope=api

# Scan hosts only
ansible-playbook aap_secure.yml -e aap_secure_scope=hosts

# With ansible-navigator
ansible-navigator run aap_secure.yml -i inventory/sample_inventory.yml -m stdout
```

## How to Use

### Option 1: Run from AAP (Recommended)

1. **Add the project** - create a Project in AAP pointing to this Git repository
2. **Add credentials**:
   - **Red Hat Ansible Automation Platform** credential (built-in type) - for API checks. This injects `aap_hostname`, `aap_username`, `aap_password`, and `aap_validate_certs` as extra vars automatically
   - **Machine** credential - for host-level and managed node checks (SSH access with become)
3. **Create a Job Template**:
   - Playbook: `aap_secure.yml`
   - Credentials: attach both credentials above
   - Extra Variables (optional):
     ```yaml
     aap_secure_mode: scan          # scan or remediate
     aap_secure_scope: api          # all, api, or hosts
     aap_secure_customer_name: "Customer Name"
     ```
4. **Run the job** - launch the template
5. **View the report**:
   - **Job stdout** - detailed per-check results and CSV output (copy-paste to spreadsheet)
   - **Artifacts tab** - structured JSON report under `aap_secure_report` key

### Option 2: Run Locally with ansible-playbook

1. **Set environment variables** for AAP API access:
   ```bash
   export AAP_HOSTNAME=https://aap-gateway.example.com/
   export AAP_USERNAME=admin
   export AAP_PASSWORD=yourpassword
   export AAP_VALIDATE_CERTS=false
   ```
   Or source from a config file (keep outside the repo):
   ```bash
   source ~/.config/aapaio
   ```
2. **Install dependencies**:
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```
3. **Run the scan**:
   ```bash
   # API checks only (no inventory needed)
   ansible-playbook aap_secure.yml -e aap_secure_scope=api

   # Full scan (requires inventory with aap_hosts and managed_nodes groups)
   ansible-playbook aap_secure.yml -i inventory/sample_inventory.yml

   # Remediate mode (applies fixes - requires write access)
   ansible-playbook aap_secure.yml -e aap_secure_mode=remediate
   ```
4. **View the report**:
   - **stdout** - per-check results and CSV output
   - **reports/** directory - HTML and JSON files

### Authentication

The playbook supports two authentication methods for API checks:

| Method | Variables | Priority |
|---|---|---|
| Token (Bearer) | `aap_token` or `AAP_TOKEN` env var | Checked first |
| Basic auth | `aap_username`/`aap_password` or `AAP_USERNAME`/`AAP_PASSWORD` env vars | Fallback |

When using the **Red Hat Ansible Automation Platform** credential type in AAP, `aap_username` and `aap_password` are injected automatically as extra vars - no additional configuration needed.

### Report Formats

| Format | Location | Best For |
|---|---|---|
| Detailed stdout | Job output / terminal | Quick review of each check |
| CSV | Job output / terminal | Copy-paste to spreadsheet |
| JSON artifact | AAP Artifacts tab | Programmatic processing |
| HTML file | `reports/` directory | Sharing with stakeholders (local runs) |
| JSON file | `reports/` directory | Archival (local runs) |

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
├── aap_secure.yml                        # Main playbook (scan + remediate)
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
