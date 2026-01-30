# Claude Code Guidelines

This file contains Claude-specific guidance.

Claude MUST follow all rules in AGENTS.md.
In case of conflict, AGENTS.md takes precedence unless explicitly stated here.

## Claude-Specific Notes

### Context Loading

When working on the VeeCode DevPortal ecosystem:

1. **Always read AGENTS.md first** — It contains the comprehensive guidance for working across all repositories
2. **Check project-specific context** — Each sibling repository may have its own CLAUDE.md or AGENTS.md with local overrides
3. **Reference this repo** — When unsure about conventions or architecture, check the relevant docs in this repository

### Working Directory Awareness

- If working in `devportal-base` — Focus on core runtime, build system, minimal plugins, image is built in GitHub Actions
- If working in `devportal-distro` — Focus on Docker builds, plugin bundling, configuration, image is built in GitHub Actions (also locally for testing)
- If working in `devportal-plugins` — Focus on plugin development, workspace isolation, Makefile commands from each workspace
- If working in `devportal-samples` — Focus on Docker Compose, environment variables, integration examples, shell scripts only when necessary

### Cross-Repository Changes

When a change spans multiple repositories:

1. Document the dependency order in your plan
2. Start from the lowest layer (plugins → base → distro → samples)
3. Note version bumps required at each layer
4. Consider backward compatibility implications

## Additional Resources

- [AGENTS.md](./AGENTS.md) — Primary guidance document
- [Architecture Overview](./architecture/overview.md) — System design
- [Glossary](./glossary.md) — Term definitions
