# devportal-plugins

Centralized plugin development monorepo for VeeCode plugins.

## Overview

| Property | Value |
| -------- | ----- |
| **Repository** | [veecode-platform/devportal-plugins](https://github.com/veecode-platform/devportal-plugins) |
| **Type** | Multi-workspace monorepo |
| **Package Manager** | Yarn 4.x |
| **NPM Namespace** | @veecode-platform |

## Purpose

The `devportal-plugins` repository provides structured workspaces for developing Backstage plugins with:

- Independent plugin workspaces (homepage, global-header, kubernetes, etc.)
- Both static and dynamic plugin formats
- Makefile-based build and release automation
- Local testing with devportal-base integration

## Related Documentation

- [Architecture Overview](../architecture/overview.md)
- [Dynamic Plugins](../architecture/dynamic-plugins.md)
- [Code Style](../conventions/code-style.md)
- [Naming Conventions](../conventions/naming.md)
- [devportal-base](./devportal-base.md)
