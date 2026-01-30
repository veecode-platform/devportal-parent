# devportal-distro

Production-ready Backstage distribution with pre-bundled plugins.

## Overview

| Property | Value |
| -------- | ----- |
| **Repository** | [veecode-platform/devportal-distro](https://github.com/veecode-platform/devportal-distro) |
| **Type** | Docker build (multi-stage) |
| **Base Image** | veecode/devportal-base |
| **Docker Image** | [veecode/devportal](https://hub.docker.com/r/veecode/devportal) |

## Purpose

The `devportal-distro` repository builds the production-ready Docker image with:

- Pre-bundled VeeCode plugins
- Default plugin configurations
- Multi-stage Docker build process
- Full distribution ready for deployment

## Related Documentation

- [Architecture Overview](../architecture/overview.md)
- [Dynamic Plugins](../architecture/dynamic-plugins.md)
- [Deployment](../architecture/deployment.md)
- [devportal-base](./devportal-base.md)
- [devportal-samples](./devportal-samples.md)
