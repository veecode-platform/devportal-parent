# Naming Conventions

This document defines naming conventions across the VeeCode DevPortal ecosystem.

## Package Naming

Follow the Backstage naming conventions as defined in:

- [Backend System Naming Patterns](https://backstage.io/docs/backend-system/architecture/naming-patterns/)
- [Frontend System Naming Patterns](https://backstage.io/docs/frontend-system/architecture/naming-patterns/)

### NPM Packages

All VeeCode packages use the `@veecode-platform` namespace:

| Pattern | Example | Purpose |
|---------|---------|---------|
| `@veecode-platform/plugin-{name}` | `@veecode-platform/plugin-kubernetes` | Static plugin |
| `@veecode-platform/plugin-{name}-dynamic` | `@veecode-platform/plugin-kubernetes-dynamic` | Dynamic plugin wrapper |
| `@veecode-platform/plugin-{name}-backend` | `@veecode-platform/plugin-kubernetes-backend` | Backend plugin |
| `@veecode-platform/plugin-{name}-common` | `@veecode-platform/plugin-kubernetes-common` | Shared types |

### VeeCode-Specific Plugins

We should generally follow the Backstage plugin naming conventions (and [Backstage ADR](https://backstage.io/docs/architecture-decisions/adrs-adr011) on this), but with a `veecode` prefix when we are authoring a plugin.

Plugins developed specifically for VeeCode (not Backstage community):

```pre
@veecode-platform/plugin-veecode-{name}
@veecode-platform/plugin-veecode-{name}-dynamic
```

Examples:

- `@veecode-platform/plugin-veecode-homepage`
- `@veecode-platform/plugin-veecode-global-header`

### Scaffolder Actions

```pre
@veecode-platform/scaffolder-backend-module-{name}
```

Examples:

- `@veecode-platform/scaffolder-backend-module-kong`

## File Naming

### TypeScript/JavaScript

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `HomePage.tsx`, `EntityCard.tsx` |
| Hooks | camelCase with `use` prefix | `useEntityData.ts` |
| Utilities | camelCase | `formatDate.ts`, `apiClient.ts` |
| Types/Interfaces | PascalCase | `types.ts` (file), `EntityData` (type) |
| Constants | UPPER_SNAKE_CASE | `constants.ts`, `API_BASE_URL` |
| Tests | Same as source + `.test` | `HomePage.test.tsx` |

### Configuration Files

| Type | Convention | Example |
|------|------------|---------|
| Backstage config | kebab-case | `app-config.yaml` |
| Profile config | `app-config.{profile}.yaml` | `app-config.github.yaml` |
| Package config | standard names | `package.json`, `tsconfig.json` |
| Docker | standard names | `Dockerfile`, `docker-compose.yml` |

### Directories

| Type | Convention | Example |
|------|------------|---------|
| Packages | kebab-case | `plugin-utils`, `dynamic-plugins-info` |
| Plugin workspaces | kebab-case | `homepage`, `global-header` |
| Source directories | lowercase | `src`, `dist`, `lib` |

## Component Naming

### React Components

```typescript
// Component file: HomePage.tsx
export const HomePage = () => { ... };

// Props interface
export interface HomePageProps {
  title: string;
}

// With props
export const HomePage = (props: HomePageProps) => { ... };
```

### Backstage Extensions

```typescript
// Plugin definition
export const myPlugin = createPlugin({
  id: 'my-plugin',  // kebab-case
  ...
});

// Route refs
export const rootRouteRef = createRouteRef({
  id: 'my-plugin',  // Match plugin id
});

// Components exposed by plugin
export const MyPluginPage = myPlugin.provide(
  createRoutableExtension({
    name: 'MyPluginPage',  // PascalCase
    ...
  })
);
```

## API Naming

### REST Endpoints

```pre
/api/{plugin-name}/{resource}
/api/{plugin-name}/{resource}/:id
/api/{plugin-name}/{resource}/:id/{sub-resource}
```

Examples:

- `/api/catalog/entities`
- `/api/kubernetes/clusters`
- `/api/kubernetes/clusters/:clusterId/pods`

### Backend Routes

```typescript
// Router registration
router.get('/entities', async (req, res) => { ... });
router.get('/entities/:id', async (req, res) => { ... });
```

## Configuration Keys

### app-config.yaml

Use camelCase for configuration keys:

```yaml
# Good
myPlugin:
  apiBaseUrl: https://api.example.com
  enableFeature: true

# Bad
my-plugin:
  api_base_url: https://api.example.com
  enable-feature: true
```

### Environment Variables

Use UPPER_SNAKE_CASE:

```bash
GITHUB_CLIENT_ID
GITHUB_CLIENT_SECRET
VEECODE_PROFILE
POSTGRES_HOST
```

## Catalog Entities

### Entity Names

```yaml
# Use lowercase with hyphens
metadata:
  name: my-service  # Good
  name: MyService   # Avoid
  name: my_service  # Avoid
```

### Entity Namespaces

```yaml
metadata:
  namespace: default  # lowercase
```

### Entity Kinds

```yaml
# Standard Backstage kinds (PascalCase)
kind: Component
kind: API
kind: Resource
kind: System
kind: Domain
kind: Group
kind: User
kind: Template
kind: Location
```

## Docker Images

### Image Names

```pre
veecode/{product}:{tag}
```

Examples:

- `veecode/devportal:latest`
- `veecode/devportal:1.2.2`
- `veecode/devportal-base:latest`

### Tags

| Tag Pattern | Purpose |
|-------------|---------|
| `latest` | Most recent stable |
| `x.y.z` | Specific version |
| `x.y` | Latest patch of minor |
| `sha-{commit}` | Specific commit (CI) |

## Git Branches

We promote a trunk-based workflow, where all changes are made on `main` and merged back as soon as possible.

### Branch Naming

| Pattern | Purpose | Example |
|---------|---------|---------|
| `main` | Primary branch | `main` |
| `work/{short-description}` | Short-lived branch for any change (feature/fix/docs/chore); merge back to `main` ASAP | `work/add-kubernetes-plugin` |
| `release/{version}` | Optional release stabilization branch (only if you cut releases from a branch) | `release/1.3.0` |
| `hotfix/{description}` | Optional urgent fix branched from the release tag/branch | `hotfix/oauth-redirect-loop` |

### Commit Prefixes

Follow Conventional Commits:

```pre
feat(plugins): add kubernetes cluster view
fix(auth): resolve OAuth callback race condition
docs(readme): update installation instructions
chore(deps): bump backstage to 1.35.0
refactor(catalog): simplify entity processor
test(api): add integration tests for catalog API
```

## Summary Table

| Element | Convention | Example |
|---------|------------|---------|
| NPM package | `@veecode-platform/plugin-{name}` | `@veecode-platform/plugin-kubernetes` |
| React component | PascalCase | `EntityCard` |
| TypeScript file | PascalCase (component) or camelCase | `EntityCard.tsx`, `utils.ts` |
| Config file | kebab-case | `app-config.yaml` |
| Directory | kebab-case | `dynamic-plugins` |
| API endpoint | kebab-case | `/api/my-plugin/resources` |
| Config key | camelCase | `apiBaseUrl` |
| Environment variable | UPPER_SNAKE_CASE | `GITHUB_CLIENT_ID` |
| Entity name | lowercase-hyphen | `my-service` |
| Docker image | lowercase | `veecode/devportal` |
| Git branch | lowercase-slash-hyphen | `work/add-feature` |
