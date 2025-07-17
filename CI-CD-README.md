# CI/CD and Release Management

This repository uses an automated CI/CD pipeline that handles different types of builds and releases efficiently.

## Overview

The CI/CD system consists of several workflows that work together:

1. **PR Experimental Builds** - Builds Docker images for every PR
2. **Release Drafter** - Automatically creates draft releases
3. **Release Publishing** - Publishes final releases and Docker images
4. **Cleanup** - Removes old experimental images

## Workflows

### 1. PR Docker Build (`.github/workflows/pr-docker-build.yml`)

**Triggers:** Pull request opened, updated, or synchronized

**What it does:**
- Builds a Docker image for each PR with a unique tag: `pr-<PR_NUMBER>-<SHORT_SHA>`
- Pushes the image to GHCR
- Comments on the PR with testing instructions
- Cleans up the image when the PR is closed

**Example tag:** `ghcr.io/tothambrus11/hylo-compiler-dev-env:pr-123-abc1234`

### 2. Release Drafter (`.github/workflows/release-drafter.yml`)

**Triggers:** 
- Push to main branch
- Pull request events (for auto-labeling)

**What it does:**
- Automatically creates and updates draft releases
- Categorizes changes based on PR labels
- Suggests version numbers based on labels
- Builds a Docker image for the draft release
- Updates the draft release with Docker image information

**Draft image tag:** `ghcr.io/tothambrus11/hylo-compiler-dev-env:latest-draft`

### 3. Release Publishing (`.github/workflows/release-publish.yml`)

**Triggers:** Release published

**What it does:**
- Builds and pushes the final Docker image with release tag
- Updates the `latest` tag
- Cleans up the draft image

**Release tags:** 
- `ghcr.io/tothambrus11/hylo-compiler-dev-env:v1.0.0`
- `ghcr.io/tothambrus11/hylo-compiler-dev-env:latest`

### 4. Cleanup Images (`.github/workflows/cleanup-images.yml`)

**Triggers:** 
- Weekly schedule (Sundays at 2 AM UTC)
- Manual trigger

**What it does:**
- Removes PR images older than 7 days
- Removes old draft images
- Keeps release images intact

## Release Process

### Automatic Release Creation

1. **Create a PR** with your changes
2. **Add appropriate labels** to the PR:
   - `feature` / `enhancement` → Minor version bump
   - `fix` / `bugfix` / `bug` → Patch version bump
   - `major` / `breaking` → Major version bump
3. **Test the PR image** using the automtic PR comment's instructions
4. **Merge the PR** to main
5. **Release Drafter** automatically updates the draft release
6. **Test the draft release** using the draft image
7. **Publish the release** when ready

### Manual Release Creation

You can also create releases manually:

1. Go to the [Releases page](../../releases)
2. Click "Draft a new release"
3. Choose or create a tag (e.g., `v1.0.0`)
4. Fill in the release notes
5. Publish the release

## Testing Images

### PR Images
```bash
# Pull and test a PR image
docker pull ghcr.io/tothambrus11/hylo-compiler-dev-env:pr-123-abc1234
docker run --rm -it ghcr.io/tothambrus11/hylo-compiler-dev-env:pr-123-abc1234
```

### Draft Release Images
```bash
# Pull and test the latest draft
docker pull ghcr.io/tothambrus11/hylo-compiler-dev-env:latest-draft
docker run --rm -it ghcr.io/tothambrus11/hylo-compiler-dev-env:latest-draft
```

### Release Images
```bash
# Pull and test a specific release
docker pull ghcr.io/tothambrus11/hylo-compiler-dev-env:v1.0.0
docker run --rm -it ghcr.io/tothambrus11/hylo-compiler-dev-env:v1.0.0

# Pull and test the latest release
docker pull ghcr.io/tothambrus11/hylo-compiler-dev-env:latest
docker run --rm -it ghcr.io/tothambrus11/hylo-compiler-dev-env:latest
```

## PR Labels

The following labels affect release notes and versioning:

### Version Bumping
- `major` / `breaking` → Major version (1.0.0 → 2.0.0)
- `minor` / `feature` / `enhancement` → Minor version (1.0.0 → 1.1.0)
- `patch` / `fix` / `bugfix` / `bug` → Patch version (1.0.0 → 1.0.1)

### Categorization
- `feature` / `enhancement` → 🚀 Features
- `fix` / `bugfix` / `bug` → 🐛 Bug Fixes
- `chore` / `maintenance` → 🧰 Maintenance
- `documentation` / `docs` → 📚 Documentation
- `ci` / `cd` / `build` → 🏗️ CI/CD
- `dependencies` / `deps` → ⬆️ Dependencies

### Exclusion
- `skip-changelog` → Excludes from release notes
- `duplicate` / `invalid` / `wontfix` → Excludes from release notes

## Image Cleanup

- **PR images** are automatically deleted when the PR is closed
- **Old experimental images** (>7 days) are cleaned up weekly
- **Draft images** are replaced when a new draft is created
- **Release images** are never automatically deleted

## Configuration

### Release Drafter Configuration

The release configuration is in `.github/release-drafter.yml`. You can customize:
- Release note templates
- Version resolution rules
- Label categorization
- Auto-labeling rules

### Workflow Configuration

Each workflow file can be customized for your needs:
- Change cleanup schedules
- Modify image retention policies
- Adjust version bump rules
- Customize release templates

## Benefits

1. **No clutter**: Experimental images are automatically cleaned up
2. **Easy testing**: Every PR and draft release has a testable Docker image
3. **Automated versioning**: Versions are determined by PR labels
4. **Organized releases**: Release notes are automatically categorized
5. **Immutable releases**: Tagged releases are never overwritten
6. **Resource efficient**: Old experimental images don't consume storage
