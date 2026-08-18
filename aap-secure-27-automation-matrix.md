# AAP Secure 2.7 - Automation Feasibility Matrix

Comprehensive mapping of every security recommendation from the Red Hat AAP Secure 2.7 guide to its automation feasibility and implementation status in `aap_secure.yml`.

**Source**: Red Hat AAP Secure 2.7 (July 2026)

**Legend**:
- **Type**: Setting = runtime config value, Config = file/system config, Best Practice = guidance/process, Architecture = design decision
- **Scan**: Yes = `aap_secure.yml` can check this in scan mode, No = requires manual review
- **Remediate**: Yes = `aap_secure.yml` can fix this in remediate mode, NA = not automatable or best practice
- **Method**: API = AAP REST API, SSH = remote host command, Installer = installer variable, Manual = human decision required, NA = not applicable
- **Status**: Done = implemented in aap_secure.yml, Planned = will implement, Not Planned = out of scope for aap_secure.yml

---

## 1. Gateway Settings

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 1 | Gateway | Set SESSION_COOKIE_SECURE=True | 198 | Setting | Yes | Yes | API | Planned | GET/PATCH /api/gateway/v1/settings/ |
| 2 | Gateway | Limit SESSIONS_PER_USER (e.g. 3-5) | 198 | Setting | Yes | Yes | API | Planned | Prevent session exhaustion |
| 3 | Gateway | Set SESSION_COOKIE_AGE (e.g. 1800s) | 198 | Setting | Yes | Yes | API | Planned | Default is too long for secure environments |
| 4 | Gateway | Configure CSRF_TRUSTED_ORIGINS | 198 | Setting | Yes | Yes | API | Planned | Must match actual gateway FQDNs |
| 5 | Gateway | Set REMOTE_HOST_HEADERS for proxy | 199 | Setting | Yes | Yes | API | Planned | Required when behind reverse proxy |
| 6 | Gateway | Configure PROXY_IP_ALLOWED_LIST | 199 | Setting | Yes | Yes | API | Planned | Restrict to known proxy IPs |
| 7 | Gateway | Disable ALLOW_OAUTH2_FOR_EXTERNAL_USERS | 67 | Setting | Yes | Yes | API | Planned | Disabled by default; verify it stays off |
| 8 | Gateway | Configure gateway route timeouts (request_timeout_seconds) | 95-96 | Setting | Yes | Yes | API | Planned | /api/gateway/v1/routes/ |
| 9 | Gateway | Configure gateway route timeouts (idle_timeout_seconds) | 95-96 | Setting | Yes | Yes | API | Planned | Prevent hung connections |
| 10 | Gateway | Verify Envoy timeout > gRPC timeout > authenticator timeout | 96 | Config | Yes | NA | API | Planned | Scan layered timeouts for consistency |
| 11 | Gateway | Use gateway_extra_settings for custom config (containerized) | 200 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 12 | Gateway | Use /etc/ansible-automation-platform/gateway/settings.py (RPM) | 200 | Config | Yes | Yes | SSH | Not Planned | Direct file edit; handled during install |

## 2. Authentication - General

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 13 | Auth | Use centralized authentication (SSO) | 7-8 | Best Practice | No | NA | NA | Not Planned | Design decision; guide strongly recommends |
| 14 | Auth | Disable local authentication for production | 10 | Setting | Yes | Yes | API | Planned | Verify no local authenticators active in prod |
| 15 | Auth | Configure authenticator maps for all authenticators | 33-40 | Config | Yes | Yes | API | Planned | Maps control org/team/role assignment |
| 16 | Auth | Use authenticator map triggers appropriately | 37-38 | Config | Yes | NA | API | Planned | Scan for "always" triggers that may be too broad |
| 17 | Auth | Set authenticator timeout < gRPC timeout | 96 | Setting | Yes | Yes | API | Planned | Prevent timeout cascade failures |
| 18 | Auth | Disable ALLOW_USER_EMAIL_SELF_EDIT | 74 | Setting | Yes | Yes | API | Planned | Deprecated; prevents account pre-hijacking |
| 19 | Auth | Run detect_changed_emails --audit | 75 | Config | Yes | NA | SSH | Planned | Management command on gateway host |

## 3. Authentication - LDAP

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 20 | LDAP | Use LDAPS (port 636) or StartTLS | 11-12 | Setting | Yes | Yes | API | Planned | Never use plain LDAP (port 389) |
| 21 | LDAP | Import CA certificate for LDAP server | 12-13 | Config | Yes | Yes | SSH | Planned | Deploy CA cert to trust store |
| 22 | LDAP | Configure LDAP bind DN with least privilege | 14 | Best Practice | Yes | NA | API | Planned | Scan bind DN permissions |
| 23 | LDAP | Set LDAP user/group search base correctly | 14-15 | Setting | Yes | Yes | API | Planned | Restrict search scope |
| 24 | LDAP | Map LDAP groups to AAP organizations/teams | 16-17 | Config | Yes | Yes | API | Planned | Via authenticator maps |

## 4. Authentication - SAML

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 25 | SAML | Enable sign_request=True | 18-19 | Setting | Yes | Yes | API | Planned | Sign SAML auth requests |
| 26 | SAML | Configure SAML Security Config properly | 19 | Setting | Yes | Yes | API | Planned | wantAssertionsSigned, wantNameIdEncrypted |
| 27 | SAML | Set GET_ALL_EXTRA_DATA for attribute mapping | 20 | Setting | Yes | Yes | API | Planned | Required for group-based authenticator maps |
| 28 | SAML | Validate SAML certificate expiry | 19 | Config | Yes | NA | API | Planned | Scan cert validity dates |

## 5. Authentication - OIDC / OAuth2

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 29 | OIDC | Configure proper JWT Algorithm (RS256 preferred) | 28 | Setting | Yes | Yes | API | Planned | Avoid HS256 for production |
| 30 | OIDC | Set appropriate scopes | 28-29 | Setting | Yes | Yes | API | Planned | Minimize requested scopes |
| 31 | OIDC | Configure IGNORE_DEFAULT_SCOPE if needed | 29 | Setting | Yes | Yes | API | Planned | Some providers need custom scope handling |
| 32 | OIDC | Use Keycloak with proper realm config | 24-27 | Config | Yes | NA | API | Planned | Scan Keycloak authenticator settings |
| 33 | OIDC | Configure Microsoft Entra ID correctly | 22-23 | Config | Yes | NA | API | Planned | Verify tenant/client configuration |
| 34 | OIDC | Configure Google OAuth2 with org restriction | 23-24 | Config | Yes | NA | API | Planned | Limit to specific Google Workspace domains |

## 6. Authentication - Other Providers

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 35 | GitHub | Restrict GitHub auth to specific org/team | 29-31 | Setting | Yes | Yes | API | Planned | GitHub org/team/enterprise variants |
| 36 | RADIUS | Use shared secret with sufficient complexity | 31-32 | Best Practice | No | NA | NA | Not Planned | Cannot scan secret strength via API |
| 37 | TACACS+ | Migrate away from TACACS+ (deprecated) | 32 | Best Practice | Yes | NA | API | Planned | Scan for active TACACS+ authenticators |

## 7. Authenticator Mapping

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 38 | AuthMap | Configure Allow maps to restrict login | 34-35 | Config | Yes | Yes | API | Planned | Control who can log in |
| 39 | AuthMap | Set Organization maps for automatic assignment | 35-36 | Config | Yes | Yes | API | Planned | Map IdP groups to AAP orgs |
| 40 | AuthMap | Set Team maps for automatic assignment | 36 | Config | Yes | Yes | API | Planned | Map IdP groups to AAP teams |
| 41 | AuthMap | Set Role maps for RBAC assignment | 36-37 | Config | Yes | Yes | API | Planned | Map IdP attributes to AAP roles |
| 42 | AuthMap | Configure Is Superuser map carefully | 37 | Config | Yes | Yes | API | Planned | Use "Never" or "Based on groups" - not "Always" |
| 43 | AuthMap | Use Jinja-like expressions for attribute-based triggers | 38-40 | Config | Yes | NA | API | Planned | Complex conditions with for_attr_value syntax |
| 44 | AuthMap | Review map ordering (priority matters) | 37 | Best Practice | Yes | NA | API | Planned | Maps evaluated in order; first match wins |

## 8. OAuth2 Token Management

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 45 | OAuth2 | Set ACCESS_TOKEN_EXPIRE_SECONDS appropriately | 65 | Setting | Yes | Yes | API | Planned | Default changed to 31,536,000s (1 year) in 2.7 |
| 46 | OAuth2 | Set REFRESH_TOKEN_EXPIRE_SECONDS | 65 | Setting | Yes | Yes | API | Planned | Default 2,628,000s (~30 days) |
| 47 | OAuth2 | Use PATs via gateway (not direct controller) | 66 | Best Practice | No | NA | NA | Not Planned | AAP 2.7 routes through platform gateway |
| 48 | OAuth2 | Run cleartokens periodically | 67-68 | Config | Yes | Yes | SSH | Planned | Management command: awx-manage cleartokens |
| 49 | OAuth2 | Run clearsessions periodically | 68 | Config | Yes | Yes | SSH | Planned | Management command: awx-manage clearsessions |
| 50 | OAuth2 | Revoke tokens for departing users | 67 | Best Practice | Yes | Yes | API | Planned | Use revoke_oauth2_tokens command or API |
| 51 | OAuth2 | Basic auth removed in 2.7 - use OAuth2 | 62-63 | Architecture | Yes | NA | API | Planned | Scan for any basic auth usage patterns |

## 9. Session Authentication

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 52 | Session | Use /api/gateway/v1/login/ for session auth | 60-61 | Best Practice | No | NA | NA | Not Planned | Requires CSRF token handling |
| 53 | Session | Include X-CSRFToken header | 61 | Best Practice | No | NA | NA | Not Planned | Required for session-based API calls |
| 54 | Session | Use SSO for browser-based access | 63-64 | Best Practice | No | NA | NA | Not Planned | Redirects through IdP |

## 10. RBAC

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 55 | RBAC | Minimize superuser accounts | 97-98 | Config | Yes | NA | API | Planned | Scan and report all superusers |
| 56 | RBAC | Use organization admins instead of superusers | 98-99 | Best Practice | Yes | NA | API | Planned | Scan org admin vs superuser ratio |
| 57 | RBAC | Assign team-level roles, not individual | 99-100 | Best Practice | Yes | NA | API | Planned | Scan for direct user role assignments |
| 58 | RBAC | Use resource-level permissions (least privilege) | 100-101 | Best Practice | Yes | NA | API | Planned | Scan for overly broad role assignments |
| 59 | RBAC | Review organization membership regularly | 98 | Best Practice | Yes | NA | API | Planned | List org members for review |
| 60 | RBAC | Restrict who can create/modify credentials | 101 | Config | Yes | NA | API | Planned | Scan credential admin role assignments |
| 61 | RBAC | Restrict who can modify inventories | 101 | Config | Yes | NA | API | Planned | Scan inventory admin assignments |

## 11. Credentials and Secrets

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 62 | Creds | Use external credential providers (CyberArk, HashiCorp, AWS SM, Azure KV) | 115-125 | Best Practice | Yes | NA | API | Planned | Scan credential types for external lookups |
| 63 | Creds | No plaintext passwords in credential fields | 127 | Config | Yes | NA | API | Planned | Cannot read values but can check credential types |
| 64 | Creds | Use Machine credential type for SSH | 102-106 | Best Practice | Yes | NA | API | Planned | Scan machine credentials exist |
| 65 | Creds | Use Network credential type for network devices | 107-109 | Best Practice | Yes | NA | API | Planned | Verify proper credential types used |
| 66 | Creds | Protect SECRET_KEY file permissions | 128-129 | Config | Yes | NA | SSH | Planned | Check /etc/tower/SECRET_KEY ownership and mode |
| 67 | Creds | Protect tower.cert/tower.key permissions | 129 | Config | Yes | NA | SSH | Planned | Check file ownership (root:awx) and mode |
| 68 | Creds | Protect postgres.py permissions | 129 | Config | Yes | NA | SSH | Planned | Contains database password |
| 69 | Creds | Protect channels.py permissions | 129 | Config | Yes | NA | SSH | Planned | Contains message bus password |
| 70 | Creds | Rotate SECRET_KEY periodically | 128 | Best Practice | No | NA | NA | Not Planned | Complex process; manual rotation only |
| 71 | Creds | Use Ansible Vault for sensitive installer variables | 203 | Best Practice | No | NA | NA | Not Planned | ansible-vault encrypt_string for passwords |
| 72 | Creds | Configure custom credential types for EDA | 173-180 | Config | Yes | Yes | API | Planned | Input config + injector config |
| 73 | Creds | Configure EDA Rule Engine credential with encryption | 181-184 | Config | Yes | Yes | API | Planned | Primary/Secondary encryption secrets for key rotation |

## 12. Credential Types Reference

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 74 | CredType | Use Azure Key Vault credential for Azure secrets | 109-111 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 75 | CredType | Use Azure Resource Manager credential type | 111-112 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 76 | CredType | Use OpenShift/K8s Bearer Token credential | 112-113 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 77 | CredType | Use OpenStack credential type | 113-114 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 78 | CredType | Use Red Hat AAP credential type | 114-115 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 79 | CredType | Use Satellite 6 credential type | 115-116 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 80 | CredType | Use Red Hat Virtualization credential type | 116-117 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 81 | CredType | Use Source Control credential type | 117-118 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 82 | CredType | Use Terraform credential type | 118-119 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 83 | CredType | Use Thycotic credential type | 119-120 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 84 | CredType | Use Ansible Vault credential type | 120-121 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 85 | CredType | Use VMware vCenter credential type | 121-122 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |
| 86 | CredType | Use Galaxy/Automation Hub credential (kind=galaxy) | 125-126 | Config | Yes | NA | API | Not Planned | Customer-specific; depends on workloads |

## 13. Content Verification

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 87 | Content | Enable GPG content verification | 126-127 | Config | Yes | Yes | API | Planned | Verify collection signatures |
| 88 | Content | Use validated/certified content from Automation Hub | 125-126 | Best Practice | Yes | NA | API | Planned | Scan project sources for trusted repos |
| 89 | Content | Configure backward-compatible API URLs | 124 | Config | Yes | NA | API | Not Planned | Environment-specific configuration |

## 14. Logging and Auditing

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 90 | Logging | Enable external log aggregator | 206-207 | Setting | Yes | Yes | API | Planned | /api/v2/settings/logging/ |
| 91 | Logging | Configure Splunk log aggregator | 206-207 | Setting | Yes | Yes | API | Planned | LOG_AGGREGATOR_HOST, PORT, PROTOCOL |
| 92 | Logging | Configure ELK log aggregator | 207 | Setting | Yes | Yes | API | Planned | Logstash/Elasticsearch target |
| 93 | Logging | Configure Sumologic log aggregator | 207 | Setting | Yes | Yes | API | Planned | HTTPS endpoint |
| 94 | Logging | Configure Loggly log aggregator | 207 | Setting | Yes | Yes | API | Planned | HTTPS endpoint |
| 95 | Logging | Verify LOG_AGGREGATOR_ENABLED=True | 206 | Setting | Yes | Yes | API | Planned | Must be enabled after configuration |
| 96 | Logging | Set appropriate LOG_AGGREGATOR_LEVEL | 207 | Setting | Yes | Yes | API | Planned | WARNING or ERROR for production |
| 97 | Logging | Enable activity stream tracking | 207 | Setting | Yes | Yes | API | Planned | ACTIVITY_STREAM_ENABLED, ACTIVITY_STREAM_ENABLED_FOR_INVENTORY_SYNC |
| 98 | Logging | Set INSIGHTS_TRACKING_STATE appropriately | 208 | Setting | Yes | Yes | API | Planned | Enable/disable Red Hat Insights data collection |

## 15. Controller Settings

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 99 | Controller | Set DEFAULT_JOB_TIMEOUT > 0 | 208 | Setting | Yes | Yes | API | Planned | Prevent runaway jobs; 0 = no timeout |
| 100 | Controller | Verify AWX_TASK_ENV has GIT_SSL_NO_VERIFY=False | 208 | Setting | Yes | Yes | API | Planned | Never disable Git SSL verification |
| 101 | Controller | Use controller_extra_settings for custom config | 200 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 102 | Controller | Use /etc/tower/conf.d/custom.py for custom settings (RPM) | 200 | Config | Yes | Yes | SSH | Not Planned | Direct file edit; handled during install |

## 16. Host Hardening - fapolicyd

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 103 | fapolicyd | Install fapolicyd package | 218-219 | Config | Yes | Yes | SSH | Planned | dnf install fapolicyd |
| 104 | fapolicyd | Enable fapolicyd service | 219 | Config | Yes | Yes | SSH | Planned | systemctl enable --now fapolicyd |
| 105 | fapolicyd | Set permissive=1 initially | 219 | Config | Yes | Yes | SSH | Planned | /etc/fapolicyd/fapolicyd.conf |
| 106 | fapolicyd | Create custom rules for ansible paths | 219-220 | Config | Yes | Yes | SSH | Planned | /etc/fapolicyd/rules.d/ allow rules |
| 107 | fapolicyd | Switch to permissive=0 (enforce) after testing | 220 | Config | Yes | Yes | SSH | Planned | Only after verifying no blocked operations |

## 17. Host Hardening - User Namespaces

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 108 | Namespaces | Set user.max_user_namespaces=65535 | 220-221 | Config | Yes | Yes | SSH | Planned | sysctl -w or /etc/sysctl.d/ |
| 109 | Namespaces | Persist sysctl setting across reboots | 221 | Config | Yes | Yes | SSH | Planned | /etc/sysctl.d/99-aap-namespaces.conf |

## 18. Host Hardening - noexec Filesystems

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 110 | noexec | Mount /tmp with noexec option | 222 | Config | Yes | Yes | SSH | Planned | Check /etc/fstab and current mounts |
| 111 | noexec | Mount /var/tmp with noexec option | 222 | Config | Yes | Yes | SSH | Planned | Check /etc/fstab and current mounts |
| 112 | noexec | Override AWX job execution path if needed | 222-223 | Setting | Yes | Yes | API | Planned | AWX_PROOT_BASE_PATH or execution path config |

## 19. Host Hardening - Log Permissions

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 113 | LogPerms | Set /var/log/nginx permissions to 770 | 223 | Config | Yes | Yes | SSH | Planned | chmod 770 /var/log/nginx |
| 114 | LogPerms | Set /var/log/tower permissions to 770 | 223 | Config | Yes | Yes | SSH | Planned | chmod 770 /var/log/tower |
| 115 | LogPerms | Set /var/log/supervisor permissions to 770 | 223 | Config | Yes | Yes | SSH | Planned | chmod 770 /var/log/supervisor |

## 20. Host Hardening - Session Timeout

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 116 | SessionTimeout | Configure StopIdleSessionSec in systemd-logind | 224 | Config | Yes | Yes | SSH | Planned | /etc/systemd/logind.conf |
| 117 | SessionTimeout | Set appropriate idle timeout value | 224 | Config | Yes | Yes | SSH | Planned | e.g. StopIdleSessionSec=900 (15 minutes) |

## 21. TLS / HSTS - NGINX

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 118 | TLS | Set *_nginx_disable_https=false | 225-226 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 119 | TLS | Set *_nginx_disable_hsts=false | 226 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 120 | TLS | Verify NGINX serves HTTPS on port 443 | 225 | Config | Yes | NA | SSH | Planned | Check nginx config and listening ports |
| 121 | TLS | Verify HSTS header present in responses | 226 | Config | Yes | NA | SSH | Planned | curl -I and check Strict-Transport-Security |

## 22. TLS / HSTS - PostgreSQL

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 122 | PgTLS | Set postgresql_disable_tls=false | 227 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 123 | PgTLS | Set *_pg_sslmode=verify-full | 227 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 124 | PgTLS | Deploy PostgreSQL CA certificate | 227-228 | Config | Yes | Yes | SSH | Planned | Required for verify-full mode |
| 125 | PgTLS | Verify PostgreSQL server certificate validity | 228 | Config | Yes | NA | SSH | Planned | Check cert expiry and chain |

## 23. PKI Certificates

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 126 | PKI | Deploy custom CA certificates | 228-229 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 127 | PKI | Use custom TLS certs for gateway | 229 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 128 | PKI | Use custom TLS certs for controller | 229 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 129 | PKI | Use custom TLS certs for hub | 229 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 130 | PKI | Use custom TLS certs for EDA | 229 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 131 | PKI | Use custom TLS certs for PostgreSQL | 230 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 132 | PKI | Use custom TLS certs for receptor | 230 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 133 | PKI | Use custom TLS certs for Redis | 230 | Config | Yes | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 134 | PKI | Monitor certificate expiry dates | 230 | Config | Yes | NA | SSH | Planned | openssl x509 -enddate on each cert |
| 135 | PKI | Verify complete CA chain | 230-231 | Config | Yes | NA | SSH | Planned | openssl verify -CAfile |
| 136 | PKI | Renew certs via installer (aap_service_regen_cert=true) | 231 | Config | No | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |
| 137 | PKI | Regenerate CA (aap_ca_regenerate=true) | 231 | Config | No | Yes | Installer | Not Planned | Installer variable; outside aap_secure.yml scope |

## 24. Managed Nodes - Service Account

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 138 | ManagedNode | Create dedicated ansible service account | 232-233 | Config | Yes | Yes | SSH | Planned | useradd ansible |
| 139 | ManagedNode | SSH key-only auth for ansible account | 233 | Config | Yes | Yes | SSH | Planned | Deploy authorized_keys |
| 140 | ManagedNode | Disable password auth for ansible account | 233 | Config | Yes | Yes | SSH | Planned | passwd -l ansible |

## 25. Managed Nodes - Sudoers

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 141 | Sudoers | Create /etc/sudoers.d/ansible with NOPASSWD | 234 | Config | Yes | Yes | SSH | Planned | ansible ALL=(ALL) NOPASSWD: ALL |
| 142 | Sudoers | Set sudoers file permissions to 0440 | 234 | Config | Yes | Yes | SSH | Planned | chmod 0440 /etc/sudoers.d/ansible |
| 143 | Sudoers | Restrict sudo commands if possible | 234 | Best Practice | Yes | NA | SSH | Planned | Limit NOPASSWD to specific commands |

## 26. Managed Nodes - SSH Daemon

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 144 | SSHd | Disable password auth for ansible user | 235 | Config | Yes | Yes | SSH | Planned | /etc/ssh/sshd_config.d/ansible.conf - PasswordAuthentication no |
| 145 | SSHd | Restrict SSH to key-based auth only | 235 | Config | Yes | Yes | SSH | Planned | Match User ansible block |
| 146 | SSHd | Restart sshd after config changes | 235 | Config | No | Yes | SSH | Planned | systemctl restart sshd |

## 27. Managed Nodes - pam_access

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 147 | pam_access | Configure pam_access for ansible user | 236 | Config | Yes | Yes | SSH | Planned | /etc/security/access.conf |
| 148 | pam_access | Restrict ansible login to controller/execution node IPs | 236 | Config | Yes | Yes | SSH | Planned | +: ansible : <controller_ips> |
| 149 | pam_access | Enable pam_access in PAM sshd config | 236 | Config | Yes | Yes | SSH | Planned | /etc/pam.d/sshd |

## 28. Firewall

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 150 | Firewall | Configure host firewall rules for AAP ports | 237 | Config | Yes | Yes | SSH | Planned | firewalld or iptables |
| 151 | Firewall | Restrict AAP API access to known networks | 237 | Config | Yes | Yes | SSH | Planned | Firewall zone configuration |
| 152 | Firewall | Allow receptor mesh ports between nodes | 237 | Config | Yes | Yes | SSH | Planned | Port 27199 by default |

## 29. Installation Security

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 153 | Install | Use dedicated installation host | 203 | Best Practice | No | NA | NA | Not Planned | Separate from AAP infrastructure |
| 154 | Install | Encrypt installer inventory with Ansible Vault | 203 | Best Practice | No | NA | NA | Not Planned | Protect passwords in inventory |
| 155 | Install | Use ansible-vault encrypt_string for individual vars | 203-204 | Best Practice | No | NA | NA | Not Planned | Fine-grained secret encryption |
| 156 | Install | Secure installer artifacts after installation | 204 | Best Practice | No | NA | NA | Not Planned | Remove or restrict access to installer directory |
| 157 | Install | Review installer inventory for exposed secrets | 203 | Best Practice | Yes | NA | SSH | Not Planned | Pre-install activity; outside aap_secure.yml scope |

## 30. Hardening Posture - Topology

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 158 | Topology | Use mesh topology for execution nodes | 198-199 | Architecture | No | NA | NA | Not Planned | Design decision for distributed execution |
| 159 | Topology | Separate control plane from execution plane | 199 | Architecture | No | NA | NA | Not Planned | Dedicated execution nodes |
| 160 | Topology | Use hop nodes for DMZ/isolated network access | 199 | Architecture | No | NA | NA | Not Planned | Receptor hop nodes |
| 161 | Topology | Avoid single points of failure | 199 | Architecture | No | NA | NA | Not Planned | HA deployment patterns |

## 31. Settings Files Security

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 162 | SettingsFiles | Protect /etc/tower/conf.d/custom.py permissions | 200 | Config | Yes | Yes | SSH | Planned | Verify ownership and mode |
| 163 | SettingsFiles | Protect /etc/ansible-automation-platform/gateway/settings.py | 200 | Config | Yes | Yes | SSH | Planned | Verify ownership and mode |
| 164 | SettingsFiles | Protect /etc/pulp/settings.py permissions | 200 | Config | Yes | Yes | SSH | Planned | Hub settings file |
| 165 | SettingsFiles | Protect /etc/ansible-automation-platform/eda/settings.yaml | 200 | Config | Yes | Yes | SSH | Planned | EDA settings file |

## 32. OpenShift Deployment

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 166 | OpenShift | Use spec.extra_settings in AnsibleAutomationPlatform CR | 201 | Config | Yes | Yes | Manual | Not Planned | OCP-specific; separate tooling |
| 167 | OpenShift | Configure Network Policies for pod isolation | 201 | Config | Yes | Yes | Manual | Not Planned | OCP-specific; separate tooling |
| 168 | OpenShift | Use Secrets for sensitive configuration | 201 | Best Practice | Yes | NA | Manual | Not Planned | OCP-specific; separate tooling |
| 169 | OpenShift | Increase Google Cloud NAT port limit if applicable | 202 | Config | No | NA | Manual | Not Planned | GCP-specific for OCP on GCP |

## 33. Security Automation Use Cases

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 170 | SecAuto | Use acl_manager role for firewall rules | 238-242 | Best Practice | No | NA | NA | Not Planned | Reference architecture |
| 171 | SecAuto | Use ids_rule for IDPS management (Snort) | 242-245 | Best Practice | No | NA | NA | Not Planned | Reference architecture |
| 172 | SecAuto | Use EDA for automated security response | 245-248 | Best Practice | No | NA | NA | Not Planned | Reference architecture |
| 173 | SecAuto | Automate patching workflows | 248-250 | Best Practice | No | NA | NA | Not Planned | Reference architecture |
| 174 | SecAuto | Monitor AWS CloudTrail with EDA | 250-252 | Best Practice | No | NA | NA | Not Planned | Reference architecture |
| 175 | SecAuto | Use F5 BIG-IP automation for WAF | 248 | Best Practice | No | NA | NA | Not Planned | Reference architecture |

## 34. Ansible Automation Portal RBAC

| # | Domain | Recommendation | Guide Page | Type | Scan | Remediate | Method | Status | Notes |
|---|--------|---------------|------------|------|------|-----------|--------|--------|-------|
| 176 | Portal | Configure catalog/scaffolder permissions | 253-254 | Config | Yes | NA | API | Planned | Control who sees automation catalog |
| 177 | Portal | Set navigation permissions per role | 255-256 | Config | Yes | NA | API | Planned | Restrict sidebar menu items |
| 178 | Portal | Use conditional access by tag | 254 | Config | Yes | NA | API | Planned | Tag-based access control |
| 179 | Portal | Review portal RBAC regularly | 253 | Best Practice | Yes | NA | API | Planned | Audit portal role assignments |
| 180 | Portal | Set synchronization frequency appropriately | 257 | Setting | Yes | Yes | API | Planned | How often portal syncs with AAP |

---

## Summary

| Category | Total | Scannable | Remediable | Status: Planned | Status: Not Planned |
|----------|-------|-----------|------------|-----------------|---------------------|
| Gateway Settings | 12 | 12 | 11 | 10 | 2 |
| Authentication - General | 7 | 6 | 4 | 6 | 1 |
| Authentication - LDAP | 5 | 5 | 4 | 5 | 0 |
| Authentication - SAML | 4 | 4 | 3 | 4 | 0 |
| Authentication - OIDC | 6 | 6 | 3 | 6 | 0 |
| Authentication - Other | 3 | 2 | 1 | 2 | 1 |
| Authenticator Mapping | 7 | 7 | 5 | 7 | 0 |
| OAuth2 Tokens | 7 | 5 | 4 | 5 | 2 |
| Session Auth | 3 | 0 | 0 | 0 | 3 |
| RBAC | 7 | 7 | 0 | 7 | 0 |
| Credentials / Secrets | 12 | 10 | 4 | 10 | 2 |
| Credential Types Reference | 13 | 13 | 0 | 0 | 13 |
| Content Verification | 3 | 3 | 1 | 2 | 1 |
| Logging / Auditing | 9 | 9 | 9 | 9 | 0 |
| Controller Settings | 4 | 4 | 4 | 2 | 2 |
| Host Hardening - fapolicyd | 5 | 5 | 5 | 5 | 0 |
| Host Hardening - Namespaces | 2 | 2 | 2 | 2 | 0 |
| Host Hardening - noexec | 3 | 3 | 3 | 3 | 0 |
| Host Hardening - Log Perms | 3 | 3 | 3 | 3 | 0 |
| Host Hardening - Session Timeout | 2 | 2 | 2 | 2 | 0 |
| TLS / HSTS - NGINX | 4 | 4 | 2 | 2 | 2 |
| TLS / HSTS - PostgreSQL | 4 | 4 | 3 | 2 | 2 |
| PKI Certificates | 12 | 10 | 10 | 2 | 10 |
| Managed Nodes - Service Account | 3 | 3 | 3 | 3 | 0 |
| Managed Nodes - Sudoers | 3 | 3 | 2 | 3 | 0 |
| Managed Nodes - SSH Daemon | 3 | 2 | 3 | 3 | 0 |
| Managed Nodes - pam_access | 3 | 3 | 3 | 3 | 0 |
| Firewall | 3 | 3 | 3 | 3 | 0 |
| Installation Security | 5 | 1 | 0 | 0 | 5 |
| Hardening Posture - Topology | 4 | 0 | 0 | 0 | 4 |
| Settings Files Security | 4 | 4 | 4 | 4 | 0 |
| OpenShift Deployment | 4 | 3 | 2 | 0 | 4 |
| Security Automation Use Cases | 6 | 0 | 0 | 0 | 6 |
| Portal RBAC | 5 | 5 | 1 | 5 | 0 |
| **TOTAL** | **180** | **142 (79%)** | **103 (57%)** | **119 (66%)** | **61 (34%)** |

---

## Automation Method Breakdown

| Method | Count | Description |
|--------|-------|-------------|
| API | 81 | AAP REST API (gateway/v1/settings, v2/settings, credentials, RBAC) |
| SSH | 48 | Remote commands on AAP hosts and managed nodes |
| Installer | 17 | AAP installer inventory variables (re-run installer to apply) |
| Manual | 5 | Requires human decision or external system access |
| NA | 29 | Best practices, architecture decisions, process guidance |

---

## Notes

- **Scan** and **Remediate** columns indicate whether `aap_secure.yml` can perform the check or fix, not just theoretical possibility.
- **Status: Planned** means the item will be implemented in `aap_secure.yml` roles. Update to **Done** as roles are built.
- **Status: Not Planned** items fall outside `aap_secure.yml` scope - installer variables, architecture decisions, OCP-specific, reference architectures, or customer-specific workload choices.
- **Installer items** require modifying the AAP installer inventory and re-running the installer - outside the scope of a post-install scan/remediate playbook.
- **Credential type items** (#74-86) are not planned because which credential types are used depends on the customer's automation workloads.
- **RBAC items** are scan-only (report current state) because the correct assignments depend on organizational structure.
- **Security automation use cases** (#170-175) are reference architectures from the guide, not configuration checks.
- This matrix covers pages 1-257 of the guide. Pages 258-276 contain permissions reference tables and legal notices with no additional security recommendations.
