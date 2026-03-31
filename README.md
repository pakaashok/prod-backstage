# 🚀 Backstage Developer Portal - Containerized

A containerized Backstage developer portal with GitHub integration for automated repository creation, team management, and user onboarding.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Build from Source](#build-from-source)
- [Configuration Files](#configuration-files)
- [Templates](#templates)
- [GitHub Integration](#github-integration)
- [Usage Guide](#usage-guide)
- [Troubleshooting](#troubleshooting)
- [Team Handoff Guide](#team-handoff-guide)

## Overview

This project provides a fully containerized Backstage developer portal that enables:

- Automated GitHub repository creation with proper CODEOWNERS
- Team creation and user onboarding via GitHub Actions
- Self-service developer workflows through a web UI

## Features

| Feature | Description |
|---------|-------------|
| 🐳 Containerized | Runs in Docker for consistency across environments |
| 🔐 Secure Secrets | PAT tokens stored in local config, not in image |
| 🏢 Multi-Org Support | Works with any GitHub organization |
| 📝 Auto CODEOWNERS | Automatically creates CODEOWNERS file in new repos |
| 👥 Team Management | Creates teams and assigns users via GitHub Actions |
| 🎨 Self-Service UI | User-friendly web interface for all operations |

## Architecture

| Layer | Technology | Purpose |
|-------|------------|---------|
| Host | Windows/Mac/Linux | Development machine |
| VM | Vagrant + Debian | Isolated environment |
| Container | Docker | Application runtime |
| Application | Backstage (Node.js 24) | Developer portal |
| Integration | GitHub API | Repository and team management |

### Workflow

1. User accesses Backstage UI at http://localhost:7007
2. User fills out template form (org, repo name, team, user)
3. Backstage creates GitHub repository with CODEOWNERS
4. Backstage triggers GitHub Actions workflow
5. Workflow creates team, invites user, grants permissions

### Port Mapping

| Layer | Port | URL |
|-------|------|-----|
| Windows Host | 7007 | http://localhost:7007 |
| Vagrant VM | 7007 | http://192.168.56.10:7007 |
| Docker Container | 7007 | Internal |

## Prerequisites

### For Using Pre-built Image

- Docker (20.10+)
- GitHub Personal Access Token with scopes: `repo`, `workflow`, `admin:org`

### For Building from Source

- Vagrant (2.3+)
- VirtualBox (7.0+)
- Git Bash (Windows) or Terminal (Mac/Linux)
- 6GB+ RAM available for VM

### GitHub Requirements

- GitHub Organization
- `platform-automation` repository with `create-team.yml` workflow
- `ORG_ADMIN_TOKEN` secret configured

## Technology Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Runtime | Node.js | 24.x |
| Package Manager | Yarn | 4.4.1 |
| Framework | Backstage | Latest |
| Container | Docker | 20.10+ |
| VM | Vagrant + VirtualBox | 2.3+ / 7.0+ |
| Base Image | node:24-trixie-slim | - |
| Database | SQLite (in-memory) | - |
| OS (VM) | Debian Bookworm | 12 |

## Project Structure
backstage/
├── Dockerfile
├── .dockerignore
├── app-config.yaml
├── app-config.local.yaml          # Gitignored - contains secrets
├── package.json
├── backstage.json
├── yarn.lock
├── .yarnrc.yml
├── .yarn/
├── packages/
│   ├── app/
│   │   └── src/
│   │       ├── App.tsx
│   │       └── modules/nav/Sidebar.tsx
│   └── backend/
│       └── src/index.ts
└── examples/
    ├── entities.yaml
    ├── org.yaml
    └── template/
        ├── template.yaml
        ├── onboard-user-template.yaml
        ├── content/
        │   ├── CODEOWNERS
        │   ├── README.md
        │   └── catalog-info.yaml
        └── onboarding-skeleton/
            ├── README.md
            └── .github/
                ├── CODEOWNERS
                └── workflows/onboard.yml