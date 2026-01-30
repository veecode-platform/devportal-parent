# Profile System

This document explains the VeeCode DevPortal profile system for authentication and catalog configuration.

## Overview

Profiles provide pre-configured authentication and catalog provider settings. A single Docker image can serve multiple deployment scenarios by selecting different profiles at runtime.

## Available Profiles

| Profile | Auth Provider | Catalog Provider | Use Case |
|---------|---------------|------------------|----------|
| `github` | GitHub OAuth | GitHub Discovery | GitHub-centric organizations |
| `azure` | Azure AD | Azure DevOps | Microsoft ecosystem |
| `ldap` | LDAP Bind | LDAP Org Sync | Enterprise with LDAP directory |
| `keycloak` | Keycloak OIDC | Keycloak Groups | Self-hosted SSO |
| `local` | Guest Auth | Static Catalog | Local development |

## Profile Selection

Set the `VEECODE_PROFILE` environment variable:

```yaml
# docker-compose.yml
services:
  devportal:
    image: veecode/devportal:latest
    environment:
      VEECODE_PROFILE: github
```

```yaml
# Kubernetes ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: devportal-config
data:
  VEECODE_PROFILE: "github"
```

## Configuration Hierarchy

Profiles work through configuration file merging:

```
┌─────────────────────────────────────────────────────────────┐
│  1. app-config.yaml (base defaults)                         │
└───────────────────────────┬─────────────────────────────────┘
                            │ merged with
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  2. app-config.{VEECODE_PROFILE}.yaml (profile settings)   │
└───────────────────────────┬─────────────────────────────────┘
                            │ merged with
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  3. app-config.production.yaml (production paths)          │
└───────────────────────────┬─────────────────────────────────┘
                            │ merged with
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  4. Mounted ConfigMaps / environment overrides             │
└─────────────────────────────────────────────────────────────┘
```

Later files override earlier ones. This allows profiles to set sensible defaults while still being customizable.

## Profile Details

### GitHub Profile

**Environment Variables:**
```bash
VEECODE_PROFILE=github
GITHUB_CLIENT_ID=xxx
GITHUB_CLIENT_SECRET=xxx
GITHUB_ORG=your-org
GITHUB_APP_ID=12345                    # Optional: for GitHub App
GITHUB_APP_PRIVATE_KEY="-----BEGIN..."  # Optional: for GitHub App
```

**Configuration (app-config.github.yaml):**
```yaml
auth:
  providers:
    github:
      development:
        clientId: ${GITHUB_CLIENT_ID}
        clientSecret: ${GITHUB_CLIENT_SECRET}

catalog:
  providers:
    github:
      organization: ${GITHUB_ORG}
      schedule:
        frequency: { minutes: 30 }
        timeout: { minutes: 3 }

integrations:
  github:
    - host: github.com
      apps:
        - appId: ${GITHUB_APP_ID}
          privateKey: ${GITHUB_APP_PRIVATE_KEY}
```

### Azure Profile

**Environment Variables:**
```bash
VEECODE_PROFILE=azure
AZURE_TENANT_ID=xxx
AZURE_CLIENT_ID=xxx
AZURE_CLIENT_SECRET=xxx
AZURE_DEVOPS_ORGANIZATION=your-org
AZURE_DEVOPS_TOKEN=xxx
```

**Configuration (app-config.azure.yaml):**
```yaml
auth:
  providers:
    microsoft:
      development:
        clientId: ${AZURE_CLIENT_ID}
        clientSecret: ${AZURE_CLIENT_SECRET}
        tenantId: ${AZURE_TENANT_ID}

catalog:
  providers:
    azureDevOps:
      organization: ${AZURE_DEVOPS_ORGANIZATION}
      project: '*'

integrations:
  azure:
    - host: dev.azure.com
      credentials:
        - organizations:
            - ${AZURE_DEVOPS_ORGANIZATION}
          personalAccessToken: ${AZURE_DEVOPS_TOKEN}
```

### LDAP Profile

**Environment Variables:**
```bash
VEECODE_PROFILE=ldap
LDAP_URL=ldaps://ldap.example.com
LDAP_BIND_DN=cn=admin,dc=example,dc=com
LDAP_BIND_SECRET=xxx
LDAP_USERS_BASE_DN=ou=users,dc=example,dc=com
LDAP_GROUPS_BASE_DN=ou=groups,dc=example,dc=com
```

**Configuration (app-config.ldap.yaml):**
```yaml
auth:
  providers:
    ldap:
      development:
        url: ${LDAP_URL}
        bindDN: ${LDAP_BIND_DN}
        bindSecret: ${LDAP_BIND_SECRET}

catalog:
  providers:
    ldapOrg:
      default:
        target: ${LDAP_URL}
        bind:
          dn: ${LDAP_BIND_DN}
          secret: ${LDAP_BIND_SECRET}
        users:
          dn: ${LDAP_USERS_BASE_DN}
        groups:
          dn: ${LDAP_GROUPS_BASE_DN}
```

### Keycloak Profile

**Environment Variables:**
```bash
VEECODE_PROFILE=keycloak
KEYCLOAK_URL=https://keycloak.example.com
KEYCLOAK_REALM=master
KEYCLOAK_CLIENT_ID=backstage
KEYCLOAK_CLIENT_SECRET=xxx
```

**Configuration (app-config.keycloak.yaml):**
```yaml
auth:
  providers:
    oidc:
      development:
        metadataUrl: ${KEYCLOAK_URL}/realms/${KEYCLOAK_REALM}/.well-known/openid-configuration
        clientId: ${KEYCLOAK_CLIENT_ID}
        clientSecret: ${KEYCLOAK_CLIENT_SECRET}

catalog:
  providers:
    keycloakOrg:
      default:
        baseUrl: ${KEYCLOAK_URL}
        realm: ${KEYCLOAK_REALM}
```

### Local Profile

**Environment Variables:**
```bash
VEECODE_PROFILE=local
# No additional secrets required
```

**Configuration (app-config.local.yaml):**
```yaml
auth:
  providers:
    guest: {}

catalog:
  locations:
    - type: file
      target: /app/catalog/*.yaml
```

## Customizing Profiles

### Overriding Profile Settings

Mount a custom config file that overrides profile defaults:

```yaml
# docker-compose.yml
services:
  devportal:
    volumes:
      - ./app-config.custom.yaml:/app/app-config.local.yaml
```

The local config is loaded last and can override any setting.

### Extending Profiles

To add integrations beyond what the profile provides:

```yaml
# app-config.custom.yaml
integrations:
  # Add GitLab alongside GitHub
  gitlab:
    - host: gitlab.com
      token: ${GITLAB_TOKEN}

catalog:
  providers:
    # Add GitLab discovery alongside GitHub
    gitlab:
      yourProviderId:
        host: gitlab.com
        orgEnabled: true
```

### Multiple Auth Providers

Profiles enable one primary auth provider, but you can enable multiple:

```yaml
# app-config.custom.yaml
auth:
  providers:
    github:
      development:
        clientId: ${GITHUB_CLIENT_ID}
        clientSecret: ${GITHUB_CLIENT_SECRET}
    google:
      development:
        clientId: ${GOOGLE_CLIENT_ID}
        clientSecret: ${GOOGLE_CLIENT_SECRET}
```

## Profile Selection Logic

The startup script (`start-base.sh`) selects configuration:

```bash
#!/bin/bash

# Default to local if not set
PROFILE="${VEECODE_PROFILE:-local}"

# Merge configurations
CONFIG_FILES="app-config.yaml"
CONFIG_FILES="$CONFIG_FILES,app-config.$PROFILE.yaml"
CONFIG_FILES="$CONFIG_FILES,app-config.production.yaml"

if [ -f "/app/app-config.local.yaml" ]; then
  CONFIG_FILES="$CONFIG_FILES,app-config.local.yaml"
fi

# Start with merged config
node packages/backend --config $CONFIG_FILES
```

## Best Practices

1. **Use environment variables for secrets** — Never hardcode credentials
2. **Start with local profile for development** — No external dependencies
3. **Test profile switching** — Verify image works with different profiles
4. **Document required variables** — List all required env vars per profile
5. **Use Kubernetes secrets** — Don't put secrets in ConfigMaps

## Troubleshooting

### Auth Not Working

1. **Check profile is set:**
   ```bash
   echo $VEECODE_PROFILE
   ```

2. **Verify env vars are set:**
   ```bash
   env | grep -E 'CLIENT_ID|CLIENT_SECRET'
   ```

3. **Check callback URLs:**
   - GitHub: `https://your-domain/api/auth/github/handler/frame`
   - Azure: `https://your-domain/api/auth/microsoft/handler/frame`

### Catalog Empty

1. **Check catalog provider config:**
   - Verify org name is correct
   - Check token has read permissions
   - Review catalog provider logs

2. **Check discovery schedule:**
   - Default is every 30 minutes
   - Trigger manual refresh in catalog UI

## Related Documentation

- [Architecture Overview](./overview.md)
- [Deployment](./deployment.md)
- [devportal-samples](../projects/devportal-samples.md) — Integration examples
