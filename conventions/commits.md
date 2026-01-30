# Commit Conventions

This document defines commit message conventions for VeeCode DevPortal projects.

## Format

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

```pre
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

## Types

| Type | Purpose | Example |
|------|---------|---------|
| `feat` | New feature | `feat(catalog): add entity search API` |
| `fix` | Bug fix | `fix(auth): resolve OAuth redirect loop` |
| `docs` | Documentation | `docs(readme): update installation steps` |
| `style` | Formatting, no code change | `style: fix indentation in config files` |
| `refactor` | Code change, no feature/fix | `refactor(api): simplify entity processor` |
| `test` | Adding/updating tests | `test(catalog): add integration tests` |
| `chore` | Maintenance tasks | `chore(deps): bump backstage to 1.35.0` |
| `perf` | Performance improvement | `perf(search): cache frequent queries` |
| `ci` | CI/CD changes | `ci: add docker build workflow` |
| `build` | Build system changes | `build: update webpack config` |
| `revert` | Revert previous commit | `revert: feat(catalog): add search` |

## Scopes

### Repository Scopes

| Repository | Scopes |
|------------|--------|
| devportal-base | `backend`, `app`, `plugins`, `docker`, `config` |
| devportal-distro | `docker`, `plugins`, `config` |
| devportal-plugins | `homepage`, `global-header`, `kubernetes`, `github-workflows`, etc. |
| devportal-samples | `github`, `azure`, `ldap`, etc. |
| devportal-parent | `docs`, `architecture`, `conventions` |

### General Scopes

- `deps` — Dependency updates
- `ci` — CI/CD configuration
- `readme` — README updates
- `api` — API changes
- `ui` — User interface changes

## Description Guidelines

- Use imperative mood ("add" not "added" or "adds")
- Start with lowercase
- No period at the end
- Keep under 72 characters
- Be specific and meaningful

### Good Examples

```pre
feat(kubernetes): add cluster pod listing
fix(auth): prevent OAuth state mismatch on slow connections
docs(profiles): document LDAP configuration options
refactor(catalog): extract entity validation to separate module
chore(deps): bump @backstage/core-plugin-api to 1.10.0
```

### Bad Examples

```pre
# Too vague
fix: bug fix

# Not imperative
feat: added new feature

# Too long
feat(catalog): add a new feature that allows users to search for entities using a complex query syntax with support for filters

# Ends with period
docs: update readme.

# Not specific
chore: updates
```

## Body

Use the body for additional context when the description isn't enough:

```pre
fix(auth): prevent OAuth state mismatch on slow connections

The OAuth state parameter was being regenerated before the callback
completed on slow connections, causing a state mismatch error.

Now we store the state in session storage and only regenerate if
the session is expired.
```

## Footer

### Breaking Changes

Use `BREAKING CHANGE:` footer for breaking changes:

```pre
feat(config): change kubernetes plugin configuration format

BREAKING CHANGE: The kubernetes.clusters format has changed.

Before:
  kubernetes:
    clusters:
      - name: production

After:
  kubernetes:
    clusterLocatorMethods:
      - type: 'config'
        clusters:
          - name: production
```

### Issue References

Reference issues in the footer:

```pre
fix(catalog): resolve entity refresh race condition

Fixes #123
Closes #456
Related to #789
```

### Co-Authors

For pair programming or contributions:

```pre
feat(homepage): add quick links widget

Co-authored-by: Jane Doe <jane@example.com>
Co-authored-by: John Smith <john@example.com>
```

## AI-Generated Commits

When using AI tools to generate commits, include co-author attribution:

```pre
feat(catalog): add bulk entity import API

Co-Authored-By: Craft Agent <agents-noreply@craft.do>
```

Or for Claude Code:

```pre
feat(catalog): add bulk entity import API

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

## Multi-Line Commit Messages

Use a heredoc for proper formatting:

```bash
git commit -m "$(cat <<'EOF'
feat(kubernetes): add namespace filtering

Added ability to filter pods by namespace in the Kubernetes
plugin. Users can now specify which namespaces to display
in their app-config.yaml.

Closes #234

Co-Authored-By: Craft Agent <agents-noreply@craft.do>
EOF
)"
```

## Squashing and Amending

### Squash Commits

When squashing a feature branch, create a single well-formatted commit:

```pre
feat(homepage): add dashboard widgets

- Add quick links widget
- Add recent entities widget
- Add system health widget
- Add responsive grid layout

Closes #100, #101, #102
```

### Amend Commits

Only amend commits that haven't been pushed:

```bash
# Add forgotten changes to last commit
git add .
git commit --amend --no-edit

# Change last commit message
git commit --amend -m "feat(catalog): add entity search"
```

## Commit Hooks

Projects use Husky for git hooks:

```json
// package.json
{
  "husky": {
    "hooks": {
      "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
    }
  }
}
```

### Commitlint Configuration

```javascript
// commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'scope-enum': [
      2,
      'always',
      ['backend', 'app', 'plugins', 'docker', 'config', 'deps', 'ci', 'docs']
    ],
  },
};
```

## Examples by Repository

### devportal-base

```pre
feat(backend): add dynamic plugin hot reload
fix(app): resolve theme flickering on load
refactor(plugins): consolidate plugin registration
chore(docker): optimize image layer caching
docs(config): document all environment variables
```

### devportal-plugins

```pre
feat(homepage): add customizable widget grid
fix(kubernetes): handle cluster connection timeout
test(global-header): add accessibility tests
chore(homepage): bump Material-UI dependency
```

### devportal-distro

```pre
feat(docker): add ARM64 image support
fix(plugins): include missing scaffolder actions
chore(docker): update base image to 1.2.2
```

### devportal-samples

```pre
feat(azure): add Azure DevOps sample
fix(github): correct OAuth callback URL
docs(ldap): add LDAP configuration example
```

## Changelog Generation

Commits following these conventions enable automatic changelog generation:

```bash
# Using conventional-changelog
npx conventional-changelog -p angular -i CHANGELOG.md -s
```

This groups commits by type and extracts breaking changes automatically.

## Related Documentation

- [Versioning Strategy](./versioning.md)
- [Code Style](./code-style.md)
