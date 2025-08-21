# Increment Semantic Version Action

A GitHub Action that automatically increments semantic versions, creates tags, and optionally updates files with the new version.

## Features

- 🚀 Increment semantic versions (patch, minor, major)
- 🏷️ Automatically create and push Git tags
- 📝 Update files with new version numbers
- 🎯 Flexible version tag prefixes (e.g., `v1.0.0` or `1.0.0`)
- 📦 Create GitHub releases automatically
- 🔧 Customizable commit messages and release notes

## Usage

### Basic Example

```yaml
name: Release

on:
  push:
    branches: [ main ]
  workflow_dispatch:
    inputs:
      version_bump:
        description: 'Version bump type'
        required: true
        default: 'patch'
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Increment version
        uses: your-username/action-increment-version@v1
        with:
          version_bump: ${{ inputs.version_bump || 'patch' }}
```

### Advanced Example with File Updates

```yaml
- name: Increment version and update package.json
  uses: your-username/action-increment-version@v1
  with:
    version_bump: minor
    tag_prefix: 'v'
    update_file: 'package.json'
    update_pattern: '"version": "[^"]*"'
    update_replacement: '"version": "{version}"'
    commit_message: 'chore: bump version to {version}'
    release_title: 'Release v{version}'
    create_release: 'true'
```

### Update Multiple Files

```yaml
- name: Increment version
  id: version
  uses: your-username/action-increment-version@v1
  with:
    version_bump: patch
    tag_prefix: 'v'

- name: Update additional files
  run: |
    NEW_VERSION="${{ steps.version.outputs.new_version }}"
    sed -i "s/VERSION = .*/VERSION = \"$NEW_VERSION\"/" setup.py
    sed -i "s/__version__ = .*/__version__ = \"$NEW_VERSION\"/" src/__init__.py
    git add setup.py src/__init__.py
    git commit -m "Update version in additional files to $NEW_VERSION" || true
    git push
```

## Inputs

| Input                | Description                                       | Required | Default                     |
|----------------------|---------------------------------------------------|----------|-----------------------------|
| `version_bump`       | Type of version bump (`patch`, `minor`, `major`)  | No       | `patch`                     |
| `tag_prefix`         | Prefix for version tags (e.g., `v` for `v1.0.0`)  | No       | ``                          |
| `update_file`        | File to update with new version                   | No       | ``                          |
| `update_pattern`     | Regex pattern to match for version replacement    | No       | ``                          |
| `update_replacement` | Replacement pattern (use `{version}` placeholder) | No       | `{version}`                 |
| `create_release`     | Create a GitHub release                           | No       | `true`                      |
| `release_title`      | Title template for GitHub release                 | No       | `Release {version}`         |
| `release_notes`      | Custom release notes                              | No       | ``                          |
| `commit_message`     | Commit message template                           | No       | `Bump version to {version}` |

## Outputs

| Output             | Description                          |
|--------------------|--------------------------------------|
| `previous_version` | The previous version number          |
| `new_version`      | The new version number               |
| `tag_name`         | The full tag name (including prefix) |

## Common Use Cases

### Node.js Projects

```yaml
- name: Update package.json version
  uses: your-username/action-increment-version@v1
  with:
    version_bump: ${{ inputs.version_bump }}
    update_file: 'package.json'
    update_pattern: '"version": "[^"]*"'
    update_replacement: '"version": "{version}"'
```

### Python Projects

```yaml
- name: Update Python version
  uses: your-username/action-increment-version@v1
  with:
    version_bump: ${{ inputs.version_bump }}
    update_file: 'setup.py'
    update_pattern: 'version="[^"]*"'
    update_replacement: 'version="{version}"'
```

### Docker Images

```yaml
- name: Increment version and build Docker image
  id: version
  uses: your-username/action-increment-version@v1
  with:
    version_bump: patch
    tag_prefix: 'v'

- name: Build and push Docker image
  run: |
    VERSION="${{ steps.version.outputs.new_version }}"
    docker build -t myapp:$VERSION .
    docker push myapp:$VERSION
```

### Configuration Files

```yaml
- name: Update README badge
  uses: your-username/action-increment-version@v1
  with:
    version_bump: minor
    update_file: 'README.md'
    update_pattern: 'revision: [0-9]+\.[0-9]+\.[0-9]+'
    update_replacement: 'revision: {version}'
```

## How It Works

1. **Get Latest Tag**: Fetches the latest Git tag or defaults to `0.0.0`
2. **Calculate New Version**: Increments the version based on semantic versioning rules
3. **Update Files**: Optionally updates specified files with the new version
4. **Create Tag**: Creates and pushes a new Git tag
5. **Create Release**: Optionally creates a GitHub release

## Requirements

- The repository must have `contents: write` permission
- Git history must be available (`fetch-depth: 0` recommended)
- For releases, `GITHUB_TOKEN` must be available

## Semantic Versioning Rules

- **Patch**: Bug fixes and small changes (`1.0.0` → `1.0.1`)
- **Minor**: New features that don't break compatibility (`1.0.0` → `1.1.0`)
- **Major**: Breaking changes (`1.0.0` → `2.0.0`)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.