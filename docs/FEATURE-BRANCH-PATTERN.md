# Feature Branch Pattern

## Overview

Spec-Kit now uses **GitFlow-compliant** feature branch patterns for better CI/CD integration.

### New Pattern (v2.0+)

```
feature/######-name-of-feature
```

**Examples**:
- `feature/000001-auth-system`
- `feature/000042-payment-gateway`
- `feature/000999-user-dashboard`

### Benefits

1. **CI/CD Automation**: CI/CD systems recognize `feature/*` patterns and can auto-deploy to Dev environments
2. **GitFlow Compliance**: Industry-standard pattern compatible with GitHub Flow, GitLab Flow, and GitFlow
3. **Better Organization**: Easily list all features with `git branch --list 'feature/*'`
4. **Branch Protection**: Apply branch protection rules to `feature/*` pattern
5. **Six-Digit Numbering**: Supports up to 999,999 features with consistent sorting

### Old Pattern (v1.x)

```
###-name-of-feature
```

**Examples**:
- `001-auth-system`
- `042-payment-gateway`
- `999-user-dashboard`

### Migration

**Existing projects** continue to work with old pattern branches. The scripts support **both patterns** simultaneously:

- Old branches: `###-name` (3-digit)
- New branches: `feature/######-name` (6-digit with prefix)

**No breaking changes** - existing projects upgrade seamlessly.

### Spec Directory Structure

**Important**: Spec directories remain clean **without** the `feature/` prefix:

```
specs/
  000001-auth-system/    # Clean directory name
    spec.md
    plan.md
    tasks.md
```

**Branch name**: `feature/000001-auth-system`
**Spec directory**: `specs/000001-auth-system/`

This separation keeps the filesystem organized while branches follow GitFlow conventions.

### CI/CD Integration Example

**GitHub Actions** (`.github/workflows/deploy-dev.yml`):

```yaml
name: Deploy to Dev

on:
  push:
    branches:
      - 'feature/**'  # Triggers on all feature branches

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extract feature number
        id: feature
        run: |
          # Extract ######-name from feature/######-name
          FEATURE=$(echo ${GITHUB_REF#refs/heads/feature/})
          echo "name=$FEATURE" >> $GITHUB_OUTPUT

      - name: Deploy to Dev
        run: |
          echo "Deploying feature/${{ steps.feature.outputs.name }} to dev-${{ steps.feature.outputs.name }}.example.com"
          # Your deployment script here
```

**GitLab CI** (`.gitlab-ci.yml`):

```yaml
deploy-dev:
  stage: deploy
  only:
    - /^feature\/.*/  # Regex for feature/* branches
  script:
    - FEATURE_NAME=${CI_COMMIT_REF_NAME#feature/}
    - echo "Deploying to dev-${FEATURE_NAME}.example.com"
    # Your deployment script
  environment:
    name: dev-$FEATURE_NAME
    url: https://dev-$FEATURE_NAME.example.com
```

### Commands Reference

All `/speckit.*` commands automatically use the new pattern:

```bash
# Creates: feature/000001-user-auth
# Spec dir: specs/000001-user-auth/
/speckit.specify Add user authentication with OAuth2 support

# Creates: feature/000002-payments
# Spec dir: specs/000002-payments/
/speckit.specify Integrate Stripe payment processing
```

### Environment Variable

The `SPECIFY_FEATURE` environment variable contains the **spec directory name** (without `feature/` prefix):

```bash
echo $SPECIFY_FEATURE
# Output: 000001-auth-system

# NOT: feature/000001-auth-system
```

This consistency ensures scripts referencing `specs/$SPECIFY_FEATURE/` work correctly.

### Backward Compatibility

Scripts detect and support both patterns:

```bash
# These all work:
git branch --list '*-auth-system'
# Old: 001-auth-system
# New: feature/000001-auth-system

# CI/CD can filter:
git branch --list 'feature/*'
# Only new pattern branches
```

### Technical Details

**Branch Number Auto-Detection**:
- Scans remote branches: `git ls-remote --heads origin`
- Checks local branches: `git branch`
- Checks spec directories: `specs/*/`
- Supports both patterns during transition
- Returns next available number

**Number Padding**:
- Old: 3 digits (`001`, `042`, `999`)
- New: 6 digits (`000001`, `000042`, `000999`)
- Consistent zero-padding ensures proper sorting

**Branch Name Length Limit**:
- GitHub enforces 244-byte limit
- Scripts auto-truncate long names
- Calculation: `feature/` (8) + number (6) + `-` (1) = 15 chars reserved
- Maximum suffix: 229 characters

---

**Last Updated**: 2025-01-26
**Version**: 2.0.0
