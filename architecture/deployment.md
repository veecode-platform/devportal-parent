# Deployment Patterns

This document covers deployment patterns for VeeCode DevPortal in various environments.

## Container Architecture

VeeCode DevPortal is distributed as Docker images:

| Image | Purpose | Base |
|-------|---------|------|
| `veecode/devportal-base` | Minimal runtime | Red Hat UBI 10 + Node.js 22 |
| `veecode/devportal` | Full distribution with plugins | `veecode/devportal-base` |

### Image Contents

```
/app/
├── packages/
│   ├── backend/             # Compiled backend
│   └── app/                 # Compiled frontend
├── dynamic-plugins/
│   └── dist/                # Pre-installed plugins
├── dynamic-plugins-root/    # Runtime plugin directory (empty)
├── app-config*.yaml         # Configuration files
├── start-base.sh           # Startup script
└── install-dynamic-plugins.sh
```

### Included Tools

The container includes operational tools:
- `yq` — YAML processing
- `jq` — JSON processing
- `kubectl` — Kubernetes CLI
- `deck` — Kong declarative config
- `mkdocs` — Documentation generation
- `python3` — Python 3.12 runtime

## Docker Compose Deployment

### Minimal Setup

```yaml
version: '3.8'

services:
  devportal:
    image: veecode/devportal:latest
    ports:
      - "7007:7007"
    environment:
      VEECODE_PROFILE: local
```

### Production Setup

```yaml
version: '3.8'

services:
  devportal:
    image: veecode/devportal:1.2.2
    ports:
      - "7007:7007"
    environment:
      VEECODE_PROFILE: github
      GITHUB_CLIENT_ID: ${GITHUB_CLIENT_ID}
      GITHUB_CLIENT_SECRET: ${GITHUB_CLIENT_SECRET}
      GITHUB_ORG: ${GITHUB_ORG}
      GITHUB_APP_ID: ${GITHUB_APP_ID}
      GITHUB_APP_PRIVATE_KEY: ${GITHUB_APP_PRIVATE_KEY}
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_USER: backstage
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - ./dynamic-plugins.yaml:/app/dynamic-plugins.yaml:ro
      - ./app-config.local.yaml:/app/app-config.local.yaml:ro
    depends_on:
      - postgres

  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: backstage
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: backstage
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### Environment File

```bash
# .env
GITHUB_CLIENT_ID=your-client-id
GITHUB_CLIENT_SECRET=your-client-secret
GITHUB_ORG=your-organization
GITHUB_APP_ID=123456
GITHUB_APP_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----"
POSTGRES_PASSWORD=secure-password
```

## Kubernetes Deployment

### Basic Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devportal
spec:
  replicas: 2
  selector:
    matchLabels:
      app: devportal
  template:
    metadata:
      labels:
        app: devportal
    spec:
      containers:
        - name: devportal
          image: veecode/devportal:1.2.2
          ports:
            - containerPort: 7007
          env:
            - name: VEECODE_PROFILE
              value: "github"
          envFrom:
            - secretRef:
                name: devportal-secrets
          volumeMounts:
            - name: config
              mountPath: /app/dynamic-plugins.yaml
              subPath: dynamic-plugins.yaml
            - name: config
              mountPath: /app/app-config.local.yaml
              subPath: app-config.local.yaml
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "2Gi"
              cpu: "1000m"
          readinessProbe:
            httpGet:
              path: /healthcheck
              port: 7007
            initialDelaySeconds: 30
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /healthcheck
              port: 7007
            initialDelaySeconds: 60
            periodSeconds: 30
      volumes:
        - name: config
          configMap:
            name: devportal-config
```

### Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: devportal
spec:
  selector:
    app: devportal
  ports:
    - port: 80
      targetPort: 7007
  type: ClusterIP
```

### Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: devportal
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - devportal.example.com
      secretName: devportal-tls
  rules:
    - host: devportal.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: devportal
                port:
                  number: 80
```

### ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: devportal-config
data:
  dynamic-plugins.yaml: |
    plugins:
      - package: "@veecode-platform/plugin-veecode-homepage-dynamic"
        disabled: false
      - package: "@veecode-platform/plugin-kubernetes"
        disabled: false
        pluginConfig:
          kubernetes:
            clusterLocatorMethods:
              - type: 'config'

  app-config.local.yaml: |
    app:
      title: My DevPortal
      baseUrl: https://devportal.example.com

    backend:
      baseUrl: https://devportal.example.com
      cors:
        origin: https://devportal.example.com
```

### Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: devportal-secrets
type: Opaque
stringData:
  GITHUB_CLIENT_ID: "your-client-id"
  GITHUB_CLIENT_SECRET: "your-client-secret"
  GITHUB_ORG: "your-organization"
  GITHUB_APP_ID: "123456"
  GITHUB_APP_PRIVATE_KEY: |
    -----BEGIN RSA PRIVATE KEY-----
    ...
    -----END RSA PRIVATE KEY-----
```

## Database Configuration

### SQLite (Development)

Default configuration, uses in-memory or file-based SQLite:

```yaml
backend:
  database:
    client: better-sqlite3
    connection: ':memory:'
```

### PostgreSQL (Production)

```yaml
backend:
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
      database: backstage
```

## Scaling Considerations

### Horizontal Scaling

DevPortal supports horizontal scaling with some considerations:

```
┌─────────────────────────────────────────────────────────────────┐
│                       Load Balancer                             │
└───────────────────────────┬─────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│    Pod 1      │   │    Pod 2      │   │    Pod 3      │
│  DevPortal    │   │  DevPortal    │   │  DevPortal    │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                            ▼
                    ┌───────────────┐
                    │  PostgreSQL   │
                    │  (Shared)     │
                    └───────────────┘
```

**Requirements:**
- Shared database (PostgreSQL)
- Session affinity for OAuth flows (or external session store)
- Shared search index (external Elasticsearch) for consistency

### Resource Sizing

| Deployment Size | Memory | CPU | Replicas |
|-----------------|--------|-----|----------|
| Development | 512Mi | 250m | 1 |
| Small (< 50 users) | 1Gi | 500m | 1-2 |
| Medium (50-200 users) | 2Gi | 1000m | 2-3 |
| Large (> 200 users) | 4Gi | 2000m | 3+ |

## Helm Chart

While not officially provided, a basic Helm structure:

```
devportal-chart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── secret.yaml
```

### values.yaml

```yaml
image:
  repository: veecode/devportal
  tag: "1.2.2"
  pullPolicy: IfNotPresent

replicaCount: 2

profile: github

ingress:
  enabled: true
  hostname: devportal.example.com
  tls: true

resources:
  requests:
    memory: "1Gi"
    cpu: "500m"
  limits:
    memory: "2Gi"
    cpu: "1000m"

postgresql:
  enabled: true
  auth:
    database: backstage
```

## Monitoring

### Health Endpoints

- `/healthcheck` — Basic health check
- `/api/status` — Detailed status (requires auth)

### Prometheus Metrics

Enable metrics endpoint:

```yaml
# app-config.local.yaml
backend:
  metrics:
    enabled: true
```

Metrics available at `/metrics`.

### Logging

Container logs to stdout/stderr:

```bash
# Docker
docker logs devportal

# Kubernetes
kubectl logs deployment/devportal -f
```

Log level configuration:

```yaml
backend:
  logger:
    level: info  # debug, info, warn, error
```

## Security Hardening

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: devportal
spec:
  podSelector:
    matchLabels:
      app: devportal
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress-nginx
      ports:
        - port: 7007
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              name: database
      ports:
        - port: 5432
    - to:  # Allow external API calls
        - ipBlock:
            cidr: 0.0.0.0/0
      ports:
        - port: 443
```

### Pod Security

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
  containers:
    - name: devportal
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: false  # Required for plugin installation
        capabilities:
          drop:
            - ALL
```

## Related Documentation

- [Architecture Overview](./overview.md)
- [Profiles](./profiles.md)
- [devportal-samples](../projects/devportal-samples.md) — Docker Compose examples
