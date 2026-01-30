# Glossary

Terms and definitions used across the VeeCode DevPortal ecosystem.

## A

### ADR (Architecture Decision Record)

A document capturing an important architectural decision along with its context and consequences. Stored in the [decisions/](./decisions/) directory.

### App Shell
The outer React application that hosts dynamic plugin modules. Uses Scalprum for module federation.

## B

### Backstage
An open-source platform for building developer portals, created by Spotify. VeeCode DevPortal is built on Backstage.

### Backend
The Node.js server component of Backstage that provides APIs, handles authentication, and runs processors.

## C

### Catalog
The central registry of all software components, systems, APIs, and resources in Backstage. Entities are stored here.

### Catalog Entity
A record in the Backstage catalog representing a software component, API, resource, user, group, or other item. Defined in `catalog-info.yaml` files.

### ConfigMap
Kubernetes resource for storing configuration data. Used to mount configuration files into DevPortal containers.

## D

### Dynamic Plugin
A Backstage plugin that can be loaded at runtime without rebuilding the application. Configured via `dynamic-plugins.yaml`.

### Dynamic Plugins Root
Directory (`/app/dynamic-plugins-root/`) where enabled plugins are installed at container startup.

## E

### Entity
See [Catalog Entity](#catalog-entity).

### Entity Ref
A string reference to a catalog entity in the format `kind:namespace/name` (e.g., `component:default/my-service`).

## F

### Frontend
The React single-page application component of Backstage that users interact with in their browser.

## I

### IDP (Internal Developer Portal)
A centralized platform for developers to discover services, documentation, and tools within an organization. DevPortal is an IDP.

### Integration
A connection to an external service (GitHub, Azure, Jenkins, etc.) that provides data to the catalog or enables features.

## K

### Kind
The type of a catalog entity. Standard kinds include Component, API, Resource, System, Domain, Group, User, Template, and Location.

## L

### Location
A catalog entity that points to a source of other entities (e.g., a GitHub repository containing `catalog-info.yaml` files).

## M

### Monorepo
A repository containing multiple packages or projects. Both devportal-base and devportal-plugins use monorepo structures.

### Mount Point
A location in the UI where dynamic plugins can inject components. Defined in plugin configuration.

## P

### Platform Engineering
The discipline of designing and building toolchains and workflows for software engineering organizations. DevPortal enables platform engineering practices.

### Plugin
A modular extension to Backstage that adds functionality. Can be frontend (UI), backend (API), or scaffolder actions.

### Profile
A pre-configured set of authentication and catalog provider settings. Selected via `VEECODE_PROFILE` environment variable.

### Provider
A component that discovers or syncs entities from external sources (e.g., GitHub provider discovers repositories).

## R

### RBAC (Role-Based Access Control)
Permission system where access is granted based on roles. Configured via `rbac-policy.csv`.

### RHDH (Red Hat Developer Hub)
Red Hat's enterprise distribution of Backstage. VeeCode DevPortal is inspired by RHDH patterns but is independent.

### Route Ref
A reference to a route in Backstage. Used for linking between plugins.

## S

### Scaffolder
The Backstage component for creating new projects from templates. Executes template actions.

### Scaffolder Action
A reusable step in a scaffolder template. Actions perform operations like creating files, making API calls, or running commands.

### Scalprum
Module federation library used by Backstage for loading dynamic frontend plugins at runtime.

### Static Plugin
A Backstage plugin compiled directly into the application bundle. Requires rebuilding to add/remove.

### System
A collection of related catalog entities (components, APIs, resources) that work together.

## T

### TechDocs
Backstage's documentation-as-code solution. Generates documentation sites from Markdown in repositories.

### Template
A scaffolder template that creates new projects. Defined in YAML with inputs and action steps.

### Turbo
Build system used in devportal-base for orchestrating monorepo tasks.

## U

### UBI (Universal Base Image)
Red Hat's container base images designed for enterprise environments. DevPortal uses UBI 10.

## V

### VeeCode DevPortal
This project — a ready-to-use, open-source Backstage distribution for Internal Developer Portals.

### VEECODE_PROFILE
Environment variable that selects which authentication profile to use (github, azure, ldap, keycloak, local).

## W

### Workspace

In `devportal-plugins`, an independent directory containing a plugin and its development environment.

In the other projects it usually refers to a yarn workspace in a monorepo project.

## Y

### YAML
Data serialization format used for Backstage configuration files (`app-config.yaml`, `catalog-info.yaml`, `dynamic-plugins.yaml`).

### Yarn
Package manager used by VeeCode DevPortal projects. Version 4.x with PnP or node-linker.

---

## Acronyms

| Acronym | Expansion |
|---------|-----------|
| API | Application Programming Interface |
| ADR | Architecture Decision Record |
| CI/CD | Continuous Integration / Continuous Deployment |
| CLI | Command Line Interface |
| CRUD | Create, Read, Update, Delete |
| IDP | Internal Developer Portal |
| LDAP | Lightweight Directory Access Protocol |
| MCP | Model Context Protocol |
| OIDC | OpenID Connect |
| RBAC | Role-Based Access Control |
| RHDH | Red Hat Developer Hub |
| SCM | Source Control Management |
| SSO | Single Sign-On |
| UBI | Universal Base Image |
| UI | User Interface |
