# Local CI/CD Pipeline Validation - HOWTO Guide

A comprehensive guide to validating your CI/CD pipelines locally before pushing
to GitLab or GitHub.

---

## Table of Contents

1. [Why Validate Locally?](#why-validate-locally)
2. [GitLab CI Local](#gitlab-ci-local)
3. [GitHub Actions with Act](#github-actions-with-act)
4. [YAML Linting with yamllint](#yaml-linting-with-yamllint)
5. [Common YAML Pitfalls](#common-yaml-pitfalls)
6. [Quick Reference](#quick-reference)

---

## Why Validate Locally?

- **Faster feedback** - No need to push and wait for CI runners
- **Save CI minutes** - Avoid burning quota on broken pipelines
- **Debug locally** - Easier to troubleshoot in your own environment
- **Learn safely** - Experiment without affecting team pipelines

---

## GitLab CI Local

### Installation

```bash
# Requires Node.js
npm install -g gitlab-ci-local

# Verify installation
gitlab-ci-local --version
```

### Basic Commands

```bash
# List all jobs in your pipeline
gitlab-ci-local --list

# List ALL jobs including when:never
gitlab-ci-local --list-all

# Preview expanded YAML (includes, extends resolved)
gitlab-ci-local --preview

# Validate dependency chain without running
gitlab-ci-local --validate-dependency-chain
```

### Running Jobs Locally

```bash
# Run the entire pipeline
gitlab-ci-local

# Run a specific job
gitlab-ci-local increment_version

# Run a specific stage
gitlab-ci-local --stage build

# Run with custom variables
gitlab-ci-local --variable MY_VAR=value

# Run jobs needed by a specific job
gitlab-ci-local --needs build_image
```

### Advanced Options

```bash
# Use a different CI file
gitlab-ci-local --file custom/.gitlab-ci.yml

# Run with timestamps for debugging
gitlab-ci-local --timestamps

# Copy artifacts to source directory
gitlab-ci-local --artifacts-to-source

# Run in privileged mode (for Docker-in-Docker)
gitlab-ci-local --privileged

# Add volumes to containers
gitlab-ci-local --volume /host/path:/container/path
```

### Variables File

Create `.gitlab-ci-local-variables.yml` for local secrets:

```yaml
# .gitlab-ci-local-variables.yml (add to .gitignore!)
DOCKER_HUB_USERNAME: your-username
DOCKER_HUB_PASSWORD: your-token
GITLAB_TOKEN: your-gitlab-token
```

---

## GitHub Actions with Act

### Installation

```bash
# macOS
brew install act

# Linux
curl https://raw.githubusercontent.com/nektos/act/master/install.sh | sudo bash

# Verify installation
act --version
```

### Basic Commands

```bash
# List all workflows and jobs
act -l

# Run default event (usually push)
act

# Dry run - validate without executing
act -n

# Run a specific job
act -j build

# Run a specific event
act pull_request
act push
act workflow_dispatch
```

### Advanced Options

```bash
# Run with specific platform image
act -P ubuntu-latest=ghcr.io/catthehacker/ubuntu:act-latest

# Pass secrets
act -s MY_SECRET=value

# Use a secrets file
act --secret-file .secrets

# Run with verbose output
act -v

# Use specific workflow file
act -W .github/workflows/ci.yml
```

### Secrets File

Create `.secrets` for local secrets:

```bash
# .secrets (add to .gitignore!)
GITHUB_TOKEN=ghp_xxxxxxxxxxxx
NPM_TOKEN=npm_xxxxxxxxxxxx
```

### Platform Images

Act uses Docker images to simulate GitHub runners:

```bash
# Micro image (smallest, fastest, limited compatibility)
act -P ubuntu-latest=node:16-buster-slim

# Medium image (good balance)
act -P ubuntu-latest=ghcr.io/catthehacker/ubuntu:act-latest

# Full image (most compatible, largest ~12GB)
act -P ubuntu-latest=ghcr.io/catthehacker/ubuntu:full-latest
```

---

## YAML Linting with yamllint

### Installation

```bash
# macOS
brew install yamllint

# Linux/pip
pip install yamllint

# Verify
yamllint --version
```

### Basic Usage

```bash
# Lint a single file
yamllint .gitlab-ci.yml

# Lint with strict mode
yamllint -s .gitlab-ci.yml

# Lint entire directory
yamllint .

# Use custom config
yamllint -c .yamllint.yml .gitlab-ci.yml
```

### Configuration File

Create `.yamllint.yml` for custom rules:

```yaml
# .yamllint.yml
extends: default

rules:
  line-length:
    max: 120          # Allow longer lines
    level: warning    # Warn instead of error
  document-start:
    present: false    # Don't require ---
  truthy:
    check-keys: false # Allow 'on:' in GitHub Actions
  comments:
    min-spaces-from-content: 1
```

---

## Common YAML Pitfalls

### 1. Colons in Strings

**Problem:** YAML interprets `:` as key-value separator

```yaml
# BAD - YAML parsing error
script:
  - echo "New version: $VERSION"

# GOOD - Quote the entire line
script:
  - 'echo "New version: $VERSION"'
```

### 2. Special Characters

**Problem:** Characters like `*`, `&`, `!`, `>`, `|`, `{`, `}`, `[`, `]`

```yaml
# BAD
script:
  - echo *.txt

# GOOD
script:
  - 'echo *.txt'
```

### 3. Boolean Values

**Problem:** YAML auto-converts yes/no/on/off to booleans

```yaml
# BAD - 'on' becomes true
on:
  push:

# GOOD - Quote if needed, or use as-is for GitHub Actions
"on":
  push:
```

### 4. Multi-line Strings

```yaml
# Literal block (preserves newlines)
script: |
  echo "Line 1"
  echo "Line 2"

# Folded block (joins lines with spaces)
description: >
  This is a long description
  that spans multiple lines.
```

### 5. Environment Variables in Different Contexts

```yaml
# GitLab CI - use ${VAR} or $VAR
script:
  - echo ${MY_VAR}
  - echo $MY_VAR

# GitHub Actions - use ${{ env.VAR }}
steps:
  - run: echo ${{ env.MY_VAR }}
```

---

## Quick Reference

### GitLab CI Local Cheat Sheet

| Command | Description |
|---------|-------------|
| `gitlab-ci-local --list` | List all jobs |
| `gitlab-ci-local --preview` | Show expanded YAML |
| `gitlab-ci-local job_name` | Run specific job |
| `gitlab-ci-local --stage test` | Run all jobs in stage |
| `gitlab-ci-local --variable X=Y` | Set variable |
| `gitlab-ci-local --needs job` | Run job with dependencies |
| `gitlab-ci-local --timestamps` | Show timing info |

### Act (GitHub Actions) Cheat Sheet

| Command | Description |
|---------|-------------|
| `act -l` | List all jobs |
| `act -n` | Dry run / validate |
| `act` | Run push event |
| `act -j build` | Run specific job |
| `act pull_request` | Run PR event |
| `act -s SECRET=val` | Pass secret |
| `act -v` | Verbose output |

### yamllint Cheat Sheet

| Command | Description |
|---------|-------------|
| `yamllint file.yml` | Lint single file |
| `yamllint -s file.yml` | Strict mode |
| `yamllint -d relaxed file.yml` | Relaxed rules |
| `yamllint -c config.yml file.yml` | Custom config |

---

## Workflow: Pre-Push Validation

Here's a recommended workflow before pushing CI changes:

```bash
# 1. Lint the YAML syntax
yamllint .gitlab-ci.yml

# 2. List jobs to verify structure
gitlab-ci-local --list

# 3. Preview expanded YAML
gitlab-ci-local --preview | head -50

# 4. Run a quick job to test
gitlab-ci-local --job my_test_job

# 5. If all passes, commit and push!
git add .gitlab-ci.yml
git commit -m "chore: Update CI pipeline"
git push
```

---

## Troubleshooting

### "script contains non string value"

Your script line contains unquoted special characters. Wrap in single quotes:

```yaml
- 'echo "Message: $VAR"'
```

### Docker-in-Docker Not Working

Use privileged mode:

```bash
gitlab-ci-local --privileged
```

### Missing Environment Variables

Create a local variables file or pass inline:

```bash
gitlab-ci-local --variable CI_PROJECT_PATH=mygroup/myproject
```

### Act: "image not found"

Pull or specify the correct platform image:

```bash
act -P ubuntu-latest=ghcr.io/catthehacker/ubuntu:act-latest
```

---

## Resources

- [gitlab-ci-local GitHub](https://github.com/firecow/gitlab-ci-local)
- [act GitHub](https://github.com/nektos/act)
- [yamllint Documentation](https://yamllint.readthedocs.io/)
- [GitLab CI/CD Reference](https://docs.gitlab.com/ee/ci/yaml/)
- [GitHub Actions Reference](https://docs.github.com/en/actions)

---

*Generated for bootcamp learning - Happy CI/CD-ing!*
