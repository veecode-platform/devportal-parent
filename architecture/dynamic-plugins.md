# Dynamic Plugin System

This document explains how the VeeCode DevPortal dynamic plugin system works.

## Overview

Dynamic plugins allow loading Backstage plugins at runtime without rebuilding the application. This enables:

- **Configuration-driven features** — Enable/disable plugins via YAML
- **No image rebuilds** — Pre-installed plugins can be toggled without rebuilding
- **External plugins** — Download plugins from NPM or OCI registries at startup
- **Plugin isolation** — Each plugin is a self-contained module

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Plugin Sources                               │
│                                                                 │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │  Pre-installed  │  │   NPM Registry  │  │  OCI Registry   │ │
│  │  (in image)     │  │   (download)    │  │  (download)     │ │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘ │
└───────────┼─────────────────────┼─────────────────────┼─────────┘
            │                     │                     │
            └─────────────────────┼─────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│              install-dynamic-plugins.sh                         │
│  • Reads dynamic-plugins.yaml                                   │
│  • Copies/downloads plugins                                     │
│  • Resolves dependencies                                        │
└─────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│              /app/dynamic-plugins-root/                         │
│  • Runtime plugin directory                                     │
│  • All enabled plugins installed here                           │
└─────────────────────────────────────────────────────────────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    ▼                           ▼
          ┌─────────────────┐         ┌─────────────────┐
          │     Backend     │         │    Frontend     │
          │  Plugin Loader  │         │    (Scalprum)   │
          └─────────────────┘         └─────────────────┘
```

## Directory Structure

### Build Time (in Docker image)

```
/app/
├── dynamic-plugins/
│   └── dist/                    # Pre-installed plugins
│       ├── veecode-homepage/
│       │   ├── package.json
│       │   └── dist/
│       └── veecode-global-header/
│           ├── package.json
│           └── dist/
└── dynamic-plugins-root/        # Empty, populated at runtime
```

### Runtime

```
/app/
├── dynamic-plugins/
│   └── dist/                    # Pre-installed source
└── dynamic-plugins-root/        # Active plugins
    ├── veecode-homepage/        # Copied from dist/
    ├── veecode-global-header/   # Copied from dist/
    └── external-plugin/         # Downloaded from NPM
```

## Configuration

### dynamic-plugins.yaml

The primary configuration file for enabling/disabling plugins:

```yaml
plugins:
  # Enable a pre-installed plugin
  - package: "@veecode-platform/plugin-veecode-homepage-dynamic"
    disabled: false

  # Disable a pre-installed plugin
  - package: "@veecode-platform/plugin-veecode-global-header-dynamic"
    disabled: true

  # Download from NPM
  - package: "@backstage/plugin-catalog-backend-module-github"
    disabled: false

  # With plugin-specific configuration
  - package: "@veecode-platform/plugin-kubernetes"
    disabled: false
    pluginConfig:
      kubernetes:
        clusterLocatorMethods:
          - type: 'config'
```

### dynamic-plugins.default.yaml

Default configuration shipped in the distro image. User's `dynamic-plugins.yaml` merges with this.

```yaml
# Defaults in image
plugins:
  - package: "@veecode-platform/plugin-veecode-homepage-dynamic"
    disabled: false
  - package: "@veecode-platform/plugin-veecode-global-header-dynamic"
    disabled: false
```

## Plugin Types

### Frontend Plugins

Frontend plugins provide UI components:

```
plugin-package/
├── package.json
├── dist/
│   ├── index.esm.js          # ES module bundle
│   ├── index.esm.js.map
│   └── alpha/                # Optional alpha exports
└── scalprum.json             # Scalprum module manifest
```

**scalprum.json example:**
```json
{
  "name": "@veecode-platform/plugin-veecode-homepage-dynamic",
  "exposedModules": {
    "PluginRoot": "./src/index.ts"
  }
}
```

### Backend Plugins

Backend plugins provide API routes and processors:

```
plugin-package/
├── package.json
├── dist/
│   ├── index.cjs.js          # CommonJS bundle
│   └── index.cjs.js.map
└── app-config.janus-idp.yaml # Plugin configuration schema
```

### Scaffolder Actions

Scaffolder actions extend template capabilities:

```
plugin-package/
├── package.json
├── dist/
│   ├── index.cjs.js
│   └── actions/              # Action implementations
└── app-config.janus-idp.yaml
```

## Plugin Loading Process

### 1. Container Startup

```bash
# entrypoint.sh
./install-dynamic-plugins.sh
./start-base.sh
```

### 2. Plugin Installation

```
┌────────────────────────────────────────────────────────────────┐
│  install-dynamic-plugins.sh                                    │
└───────────────────────────┬────────────────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            ▼                               ▼
┌───────────────────────┐       ┌───────────────────────┐
│  Read plugin configs  │       │  Check pre-installed  │
│  dynamic-plugins.yaml │       │  /app/dynamic-plugins │
│  + defaults           │       │  /dist/               │
└───────────┬───────────┘       └───────────┬───────────┘
            │                               │
            └───────────────┬───────────────┘
                            ▼
┌────────────────────────────────────────────────────────────────┐
│  For each enabled plugin:                                      │
│  1. If pre-installed → copy to dynamic-plugins-root           │
│  2. If external → npm pack/download → extract to root         │
│  3. Resolve dependencies                                       │
└────────────────────────────────────────────────────────────────┘
```

### 3. Backend Discovery

```typescript
// Backend scans dynamic-plugins-root at startup
const plugins = await discoverDynamicPlugins({
  root: '/app/dynamic-plugins-root',
});

// Each plugin's package.json contains:
// - backstage.role: 'backend-plugin' | 'frontend-plugin'
// - main: entry point
```

### 4. Frontend Loading (Scalprum)

```typescript
// Scalprum loads frontend modules dynamically
const remoteModules = await loadRemoteModules({
  manifest: '/api/dynamic-plugins/manifests',
});

// Modules registered as federated modules
// Routes and components injected into app
```

## Creating Dynamic Plugins

### From Existing Static Plugin

1. Create a wrapper package:
   ```
   dynamic-plugins/
   └── wrappers/
       └── my-plugin-dynamic/
           ├── package.json
           └── src/
               └── index.ts
   ```

2. Configure package.json:
   ```json
   {
     "name": "@veecode-platform/plugin-my-plugin-dynamic",
     "version": "1.0.0",
     "backstage": {
       "role": "frontend-plugin"
     },
     "main": "dist/index.esm.js",
     "scripts": {
       "build": "backstage-cli package build",
       "export-dynamic": "janus-cli export-dynamic-plugin"
     }
   }
   ```

3. Build and export:
   ```bash
   yarn build
   yarn export-dynamic
   ```

### Plugin Configuration Schema

Plugins can declare configuration requirements in `app-config.janus-idp.yaml`:

```yaml
dynamicPlugins:
  frontend:
    my-plugin:
      mountPoints:
        - mountPoint: 'entity.page.overview/cards'
          importName: MyPluginCard
          config:
            layout:
              gridColumn: '1 / -1'
      routeBindings:
        targets:
          - importName: myPluginRouteRef
        bindings:
          - bindTarget: 'catalogPlugin.externalRoutes'
            bindMap:
              viewMyPlugin: 'myPluginRouteRef'
```

## Troubleshooting

### Plugin Not Loading

1. **Check configuration:**
   ```bash
   cat /app/dynamic-plugins.yaml
   # Verify disabled: false
   ```

2. **Check installation:**
   ```bash
   ls -la /app/dynamic-plugins-root/
   # Verify plugin directory exists
   ```

3. **Check logs:**
   ```bash
   # Backend logs show loading errors
   kubectl logs deployment/devportal -c backend
   ```

### Plugin Loading But Not Visible

1. **Frontend plugin:** Check browser console for Scalprum errors
2. **Verify mount points:** Plugin may load but not be mounted
3. **Check route bindings:** Routes may not be connected

### Dependencies Missing

```bash
# Check package.json dependencies
cat /app/dynamic-plugins-root/my-plugin/package.json

# Shared dependencies come from base image
# Plugin-specific deps must be bundled
```

## Best Practices

1. **Bundle all dependencies** — Don't rely on base image node_modules
2. **Use peer dependencies sparingly** — Only for Backstage core packages
3. **Test with dynamic loading** — Static builds behave differently
4. **Version your plugins** — Semantic versioning for compatibility
5. **Document configuration** — Users need to know available options

## Related Documentation

- [Architecture Overview](./overview.md)
- [AGENTS.md](../AGENTS.md) — Plugin development workflow
- [devportal-plugins](../projects/devportal-plugins.md) — Plugin repository reference
