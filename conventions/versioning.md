# Versioning Strategy

This document defines the versioning strategy for VeeCode DevPortal projects.

## Semantic Versioning

All packages follow [Semantic Versioning 2.0.0](https://semver.org/):

```pre
MAJOR.MINOR.PATCH
```

| Version Part | When to Increment |
|--------------|-------------------|
| **MAJOR** | Breaking changes (API changes, config format changes, removed features) |
| **MINOR** | New features (backward-compatible additions) |
| **PATCH** | Bug fixes (backward-compatible fixes) |

## Version Alignment

### Plugins

Plugins are versioned independently:

```pre
@veecode-platform/plugin-veecode-homepage: 1.0.1
@veecode-platform/plugin-veecode-global-header: 1.0.2
@veecode-platform/plugin-kubernetes: 0.5.0
```

Plugin versions are independent of base/distro versions. A new image release will pick its bundled plugins as part of its build process.

### Docker Tags

Docker images follow the same version as their source:

| Image | Tag Examples |
|-------|--------------|
| `veecode/devportal-base` | `1.2.2`, `1.2`, `latest` |
| `veecode/devportal` | `1.2.5`, `1.2`, `latest` |

**Note:** The `devportal` image is built from the `devportal-base` base image, but they DO NOT share version numbers.

## Release Process

The release process for each container image is handled by its own repository.

The release process for each plugin is handled by its own workspace in the plugins shared repository.

## Changelog Format

Try to follow [Keep a Changelog](https://keepachangelog.com/) guidelines.

### Compatibility Matrix

We do not maintain a compatibility matrix in documentation. That may be done in the future.

## Related Documentation

- [Commit Conventions](./commits.md)
- [AGENTS.md](../AGENTS.md) — Release workflow
