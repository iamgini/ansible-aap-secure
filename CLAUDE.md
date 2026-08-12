# CLAUDE.md — ansible-aap-secure

## Project Purpose

Scan-and-remediate toolkit for AAP 2.7 security hardening based on the Red Hat "AAP Secure 2.7" guide. Designed for customer engagements — replaces a 3-4 day manual security assessment with automated scanning and remediation.

**Two modes:**
- **scan** — read-only checks, produces a compliance report (current vs recommended values)
- **remediate** — applies security fixes based on baseline + customer overrides

**Two target types:**
- **api** — talks to AAP API (gateway settings, authentication, RBAC, credentials, logging)
- **hosts** — talks to RHEL hosts via SSH (fapolicyd, namespaces, noexec, TLS, SSH hardening, managed nodes)

## Project Structure

```
ansible-aap-secure/
├── CLAUDE.md
├── ansible.cfg
├── ansible-navigator.yml
├── requirements.yml
├── scan.yml                        # Main playbook (scan + remediate, controlled by variables)
├── roles/
│   ├── gateway_security/           # Gateway settings: session, CSRF, proxy, OAuth2
│   ├── authentication/             # Authenticators: LDAP, SAML, OIDC, Keycloak config checks
│   ├── rbac/                       # RBAC: superuser audit, org/team/role assignments
│   ├── credentials/                # Credential hygiene: no_log, external vaults, plaintext detection
│   ├── logging/                    # Log aggregator: Splunk/ELK config, activity stream
│   ├── host_hardening/             # Host-level: fapolicyd, namespaces, noexec, log perms, session timeout
│   ├── tls_certificates/           # TLS/HSTS: certificate validation, NGINX config, PostgreSQL SSL
│   ├── managed_nodes/              # Managed node: service account, SSH keys, sudoers, pam_access
│   └── report/                     # Aggregate results and generate compliance report
├── vars/
│   ├── security_baseline.yml       # Default hardened values (the "gold standard" from AAP Secure guide)
│   ├── customer_overrides.yml      # Customer-specific exceptions and values
│   └── scan_policy.yml             # Which checks to enable/skip
├── templates/
│   └── compliance_report.html.j2   # HTML report template
├── inventory/
│   └── sample_inventory.yml        # Example inventory showing host groups
└── reports/                        # Generated reports (gitignored)
```

## Key Variables

```yaml
# Mode control
aap_secure_mode: scan              # scan | remediate

# Scope control
aap_secure_scope: all              # all | api | hosts

# Individual domain toggles (all default true)
aap_secure_check_gateway: true
aap_secure_check_auth: true
aap_secure_check_rbac: true
aap_secure_check_credentials: true
aap_secure_check_logging: true
aap_secure_check_host_hardening: true
aap_secure_check_tls: true
aap_secure_check_managed_nodes: true
```

## Playbook Design

Single `scan.yml` with multiple plays:

1. **Play 1** — targets `aap_gateway` (API checks): gateway settings, authentication, RBAC, credentials, logging
2. **Play 2** — targets `aap_hosts` (SSH checks): fapolicyd, namespaces, noexec, log permissions, TLS
3. **Play 3** — targets `managed_nodes` (SSH checks): service account, SSH config, sudoers, pam_access
4. **Play 4** — targets `localhost`: aggregate results, generate report

Each role internally handles both scan and remediate based on `aap_secure_mode`.

## Security Domains (from AAP Secure 2.7 Guide)

### API-Level (via AAP REST API)

| Domain | Key Checks |
|---|---|
| **Gateway Settings** | SESSION_COOKIE_SECURE, SESSIONS_PER_USER, SESSION_COOKIE_AGE, CSRF_TRUSTED_ORIGINS, REMOTE_HOST_HEADERS, PROXY_IP_ALLOWED_LIST, ALLOW_OAUTH2_FOR_EXTERNAL_USERS |
| **Authentication** | Authenticator type (LDAP→LDAPS, SAML signed, OIDC proper scopes), authenticator maps, local auth disabled for prod |
| **RBAC** | No unnecessary superusers, org admin assignments, team roles, resource-level permissions |
| **Credentials** | External vault integration (CyberArk/HashiCorp/AWS/Azure), no plaintext passwords, no_log on sensitive fields |
| **Logging** | Log aggregator enabled + connected (Splunk/ELK/Sumologic), activity stream enabled, log level appropriate |
| **Controller Settings** | Job timeouts set (not 0), AWX_TASK_ENV (GIT_SSL_NO_VERIFY=False), INSIGHTS_TRACKING_STATE |

### Host-Level (via SSH to RHEL hosts)

| Domain | Key Checks |
|---|---|
| **fapolicyd** | Installed, permissive=1, custom rules for ansible paths in /etc/fapolicyd/rules.d/ |
| **User Namespaces** | user.max_user_namespaces=65535 in sysctl |
| **noexec Filesystems** | /tmp and /var/tmp mount options, AWX job execution path override if needed |
| **Log Permissions** | /var/log/nginx, /var/log/tower, /var/log/supervisor — chmod 770 |
| **Session Timeout** | systemd-logind StopIdleSessionSec configured |
| **TLS/HSTS** | NGINX HTTPS enabled, HSTS headers, PostgreSQL sslmode=verify-full, custom CA deployed |
| **PKI Certificates** | Valid certs on gateway, controller, hub, EDA; expiry check; CA chain complete |

### Managed Node (via SSH to managed hosts)

| Domain | Key Checks |
|---|---|
| **Service Account** | Dedicated `ansible` user exists, SSH key auth only, no password |
| **Sudoers** | /etc/sudoers.d/ansible with NOPASSWD, chmod 0440 |
| **SSH Daemon** | PasswordAuthentication no for ansible user in sshd_config.d |
| **pam_access** | Restrict ansible account to controller/execution node IPs |

## AAP API Access

Uses the `ansible.platform` or `infra.aap_configuration` collections for API interaction. Requires:
- `aap_hostname` — Gateway URL
- `aap_username` / `aap_password` or `aap_oauthtoken` — Authentication
- `aap_validate_certs` — TLS verification (default: true)

For scan mode, only read API access is needed. For remediate mode, write access is required.

## Report Output

The compliance report shows per-check:
- **Item name** and description
- **Status**: PASS / FAIL / WARNING / SKIPPED
- **Current value** vs **Recommended value**
- **Remediation guidance** (what would be changed in remediate mode)
- **Reference** to AAP Secure guide section

Summary shows overall compliance percentage and breakdown by domain.

## Dependencies

```yaml
# requirements.yml
collections:
  - name: ansible.platform       # AAP 2.5+ API modules
  - name: infra.aap_configuration # CaC filetree modules
  - name: ansible.builtin        # Core modules
```

## Reference

- Source guide: AAP Secure 2.7 (Red Hat, July 2026)
- PDF location: `/home/gmadappa/Downloads/aap-secure-2-7-pdf.pdf`
- Related project: `ansible-aap-cac` (CaC filetree for AAP configuration)
