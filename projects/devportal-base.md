# devportal-base

Minimal, production-ready (almost) Backstage runtime foundation.

## Overview

| Property | Value |
| -------- | ----- |
| **Repository** | [veecode-platform/devportal-base](https://github.com/veecode-platform/devportal-base) |
| **Type** | Monorepo (Yarn workspaces + Turbo) |
| **Package Manager** | Yarn 4.12.0 |
| **Node.js** | 20 or 22 |
| **Docker Image** | [veecode/devportal-base](https://hub.docker.com/r/veecode/devportal-base) |

## Purpose

The `devportal-base` repository provides the core Backstage runtime with:

- Backend and frontend applications
- Dynamic plugin loading infrastructure
- Profile-based configuration system
- Base Docker image (used by distribution)

## Related Documentation

- [Architecture Overview](../architecture/overview.md)
- [Dynamic Plugins](../architecture/dynamic-plugins.md)
- [Profiles](../architecture/profiles.md)
- [devportal-distro](./devportal-distro.md)
