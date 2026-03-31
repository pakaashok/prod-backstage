cd ~/backstage

cat > README.md << 'EOF'
# 🚀 Backstage Developer Portal - Containerized

A containerized Backstage developer portal with GitHub integration for automated repository creation, team management, and user onboarding.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Setup Guide](#setup-guide)
  - [Infrastructure Setup](#infrastructure-setup)
  - [Build from Source](#build-from-source)
  - [Using Pre-built Image](#using-pre-built-image)
- [Configuration](#configuration)
- [Templates](#templates)
- [GitHub Integration](#github-integration)
- [Usage Guide](#usage-guide)
- [Troubleshooting](#troubleshooting)
- [Team Handoff Guide](#team-handoff-guide)

---

## Overview

This project provides a fully containerized Backstage developer portal that enables:
- **Automated GitHub repository creation** with proper CODEOWNERS
- **Team creation and user onboarding** via GitHub Actions
- **Self-service developer workflows** through a web UI

---

## Architecture


┌─────────────────────────────────────────────────────────────────────────┐
│                           WINDOWS HOST                                   │
│                                                                          │
│  ┌────────────────────────────────────────────────────────────────────┐ │
│  │                      VAGRANT VM (Debian)                           │ │
│  │                                                                    │ │
│  │  ┌──────────────────────────────────────────────────────────────┐ │ │
│  │  │                 DOCKER CONTAINER                              │ │ │
│  │  │                                                               │ │ │
│  │  │  ┌─────────────────────────────────────────────────────────┐ │ │ │
│  │  │  │              BACKSTAGE (Node.js 24)                     │ │ │ │
│  │  │  │                                                         │ │ │ │
│  │  │  │  • Frontend (React)                                     │ │ │ │
│  │  │  │  • Backend (Node.js)                                    │ │ │ │
│  │  │  │  • Scaffolder Plugin                                    │ │ │ │
│  │  │  │  • Catalog Plugin                                       │ │ │ │
│  │  │  │  • GitHub Integration                                   │ │ │ │
│  │  │  └─────────────────────────────────────────────────────────┘ │ │ │
│  │  │                         │                                     │ │ │
│  │  │                         │ Port 7007                           │ │ │
│  │  └─────────────────────────│─────────────────────────────────────┘ │ │
│  │                            │                                       │ │
│  └────────────────────────────│───────────────────────────────────────┘ │
│                               │                                         │
│                               ▼                                         │
│                    http://localhost:7007                                │
└─────────────────────────────────────────────────────────────────────────┘
│
│ GitHub API
▼
┌───────────────────────┐
│       GITHUB          │
│                       │
│  • Repository Creation│
│  • Team Management    │
│  • Actions Workflows  │
│  • CODEOWNERS         │
└───────────────────────┘

yaml

Copy code

---

## Features

| Feature | Description |
|---------|-------------|
| 🐳 Containerized | Runs in Docker for consistency across environments |
| 🔐 Secure Secrets | PAT tokens stored in local config, not in image |
| 🏢 Multi-Org Support | Works with any GitHub organization |
| 📝 Auto CODEOWNERS | Automatically creates CODEOWNERS file in new repos |
| 👥 Team Management | Creates teams and assigns users via GitHub Actions |
| 🎨 Self-Service UI | User-friendly web interface for all operations |

---

## Prerequisites

### For Building from Source
- [Vagrant](https://www.vagrantup.com/downloads) (2.3+)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (7.0+)
- Git Bash (Windows) or Terminal (Mac/Linux)
- 6GB+ RAM available for VM

### For Using Pre-built Image
- Docker (20.10+)
- GitHub Personal Access Token

### GitHub Requirements
- GitHub Organization
- PAT with scopes: `repo`, `workflow`, `admin:org`, `delete_repo`
- `platform-automation` repository with `create-team.yml` workflow

---

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

---

## Project Structure


backstage/
├── Dockerfile                    # Multi-stage Docker build
├── .dockerignore                 # Docker build exclusions
├── app-config.yaml               # Main Backstage configuration
├── app-config.local.yaml         # Local secrets (gitignored)
├── package.json                  # Root package dependencies
├── backstage.json                # Backstage metadata
├── yarn.lock                     # Dependency lock file
├── .yarnrc.yml                   # Yarn configuration
├── .yarn/                        # Yarn releases and plugins
│
├── packages/
│   ├── app/                      # Frontend application
│   │   └── src/
│   │       ├── App.tsx           # Main app with plugins
│   │       └── modules/
│   │           └── nav/
│   │               └── Sidebar.tsx
│   │
│   └── backend/                  # Backend application
│       └── src/
│           └── index.ts
│
└── examples/
├── entities.yaml             # Sample catalog entities
├── org.yaml                  # Users and groups
│
└── template/
├── template.yaml                    # Simple repo creation template
├── onboard-user-template.yaml       # Full onboarding template
│
├── content/                         # Skeleton for template 1
│   ├── CODEOWNERS
│   ├── README.md
│   └── catalog-info.yaml
│
└── onboarding-skeleton/             # Skeleton for template 2
├── README.md
└── .github/
├── CODEOWNERS
└── workflows/
└── onboard.yml

yaml

Copy code

---

## Setup Guide

### Infrastructure Setup

#### Step 1: Create Vagrantfile

```bash
mkdir -p ~/ubuntu
cd ~/ubuntu

cat > Vagrantfile << 'VAGRANTFILE'
Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  # Network configuration
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.network "forwarded_port", guest: 7007, host: 7007
  config.vm.network "forwarded_port", guest: 3000, host: 3000

  # Resources
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "6144"
    vb.cpus = 2
  end

  # Provisioning
  config.vm.provision "shell", inline: <<-SHELL
    set -e
    export DEBIAN_FRONTEND=noninteractive

    apt-get update
    apt-get upgrade -y

    # Install Docker
    curl -fsSL https://get.docker.com | sh
    usermod -aG docker vagrant
    systemctl enable docker
    systemctl start docker

    echo "Setup completed!"
  SHELL
end
VAGRANTFILE

Step 2: Start VM
bash

Copy code
vagrant up
vagrant ssh

Build from Source
Step 1: Create Backstage App (inside VM)
bash

Copy code
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Create Backstage app
npx @backstage/create-app@latest
cd backstage

Step 2: Create Dockerfile
bash

Copy code
cat > Dockerfile << 'EOF'
# Stage 1 - Create yarn install skeleton layer
FROM node:24-trixie-slim AS packages

WORKDIR /app
COPY backstage.json package.json yarn.lock ./
COPY .yarn ./.yarn
COPY .yarnrc.yml ./
COPY packages packages

RUN find packages \! -name "package.json" -mindepth 2 -maxdepth 2 -exec rm -rf {} \+

# Stage 2 - Install dependencies and build packages
FROM node:24-trixie-slim AS build

ENV PYTHON=/usr/bin/python3

RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential libsqlite3-dev && \
    rm -rf /var/lib/apt/lists/*

USER node
WORKDIR /app

COPY --from=packages --chown=node:node /app .
RUN yarn install --immutable

COPY --chown=node:node . .

RUN yarn tsc
RUN yarn --cwd packages/backend build

RUN mkdir packages/backend/dist/skeleton packages/backend/dist/bundle \
    && tar xzf packages/backend/dist/skeleton.tar.gz -C packages/backend/dist/skeleton \
    && tar xzf packages/backend/dist/bundle.tar.gz -C packages/backend/dist/bundle

# Stage 3 - Build the actual backend image
FROM node:24-trixie-slim

ENV PYTHON=/usr/bin/python3

RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential libsqlite3-dev && \
    rm -rf /var/lib/apt/lists/*

USER node
WORKDIR /app

COPY --from=build --chown=node:node /app/.yarn ./.yarn
COPY --from=build --chown=node:node /app/.yarnrc.yml ./
COPY --from=build --chown=node:node /app/backstage.json ./
COPY --from=build --chown=node:node /app/yarn.lock /app/package.json /app/packages/backend/dist/skeleton/ ./

RUN yarn workspaces focus --all --production && rm -rf "$(yarn cache clean)"

COPY --from=build --chown=node:node /app/packages/backend/dist/bundle/ ./

# Copy config and examples
COPY --chown=node:node app-config.yaml ./
COPY --chown=node:node examples ./examples

ENV NODE_ENV=production
ENV NODE_OPTIONS="--no-node-snapshot"

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
EOF

Step 3: Create .dockerignore
bash

Copy code
cat > .dockerignore << 'EOF'
dist-types
node_modules
packages/*/dist
packages/*/node_modules
*.local.yaml
.env
.git
EOF

Step 4: Build Image
bash

Copy code
docker build -t backstage .

Using Pre-built Image
Step 1: Pull Image
bash

Copy code
docker pull adeeashok/backstage:v1.2.0

Step 2: Create Local Config
bash

Copy code
cat > app-config.local.yaml << 'EOF'
integrations:
  github:
    - host: github.com
      token: ghp_YOUR_PAT_TOKEN_HERE
EOF

Step 3: Run Container
bash

Copy code
docker run -it -p 7007:7007 \
  -v $(pwd)/app-config.local.yaml:/app/app-config.local.yaml:ro \
  adeeashok/backstage:v1.2.0 \
  node packages/backend --config app-config.yaml --config app-config.local.yaml

Step 4: Access UI
javascript

Copy code
http://localhost:7007

Configuration
app-config.yaml (Main Config)
yaml

Copy code
app:
  title: Backstage Dev Portal
  baseUrl: http://localhost:7007

organization:
  name: Your Organization

backend:
  baseUrl: http://localhost:7007
  listen:
    port: 7007
    host: 0.0.0.0
  csp:
    connect-src: ["'self'", "http:", "https:"]
  cors:
    origin: http://localhost:7007
    methods: [GET, HEAD, PATCH, POST, PUT, DELETE]
    credentials: true
  database:
    client: better-sqlite3
    connection: ':memory:'

integrations:
  github:
    - host: github.com
      token: ${GITHUB_TOKEN}

auth:
  environment: development
  providers:
    guest:
      dangerouslyAllowOutsideDevelopment: true

catalog:
  rules:
    - allow: [Component, System, API, Resource, Location, Template]
  locations:
    - type: file
      target: ./examples/entities.yaml
    - type: file
      target: ./examples/template/template.yaml
      rules:
        - allow: [Template]
    - type: file
      target: ./examples/template/onboard-user-template.yaml
      rules:
        - allow: [Template]
    - type: file
      target: ./examples/org.yaml
      rules:
        - allow: [User, Group]

app-config.local.yaml (Secrets - DO NOT COMMIT)
yaml

Copy code
integrations:
  github:
    - host: github.com
      token: ghp_YOUR_ACTUAL_TOKEN

Templates
Template 1: Create Application Repo
Purpose: Creates a GitHub repository with CODEOWNERS file.

Inputs:

Field	Description
GitHub Organization	Your GitHub org name
Repository Name	Name for the new repo
Team Name	Team to assign as code owners
Steps:

Fetch template skeleton files
Create GitHub repository
Push CODEOWNERS and README
Template 2: Onboard User to Org
Purpose: Full onboarding workflow - creates repo, team, and assigns user.

Inputs:

Field	Description
GitHub Organization	Your GitHub org name
GitHub Username	User to onboard
Repository Name	Name for the new repo
Team Name	Team to create and assign
Steps:

Fetch onboarding skeleton files
Create GitHub repository
Trigger GitHub Actions workflow
Workflow creates team, invites user, grants access
GitHub Integration
Required GitHub Actions Workflow
Create this file in your org's platform-automation repository:

.github/workflows/create-team.yml

yaml

Copy code
name: Create GitHub Team

on:
  workflow_dispatch:
    inputs:
      team_name:
        description: "Team name"
        required: true
      github_user:
        description: "GitHub username"
        required: true
      repo_name:
        description: "Repository name"
        required: true

jobs:
  create-team:
    runs-on: ubuntu-latest
    steps:
      - name: Create Team, Invite User, Add to Team, and Grant Repo Access
        uses: actions/github-script@v7
        with:
          github-token: ${{ secrets.ORG_ADMIN_TOKEN }}
          script: |
            const org = context.payload.repository.owner.login;
            const teamName = context.payload.inputs.team_name;
            const username = context.payload.inputs.github_user;
            const repoName = context.payload.inputs.repo_name;

            const teamSlug = teamName
              .toLowerCase()
              .replace(/[^a-z0-9\s-]/g, '')
              .trim()
              .replace(/\s+/g, '-');

            // Step 1: Invite user to org
            try {
              const user = await github.rest.users.getByUsername({ username });
              await github.rest.orgs.createInvitation({
                org,
                invitee_id: user.data.id,
                role: "direct_member",
              });
              core.info(`📨 Invitation sent to ${username}`);
            } catch (error) {
              if (error.status === 422) {
                core.info(`⚠️ User already invited or in org`);
              } else {
                throw error;
              }
            }

            // Step 2: Create team
            try {
              await github.rest.teams.create({
                org,
                name: teamName,
                privacy: "closed",
              });
              core.info(`✅ Team created: ${teamName}`);
            } catch (error) {
              if (error.status === 422) {
                core.info(`⚠️ Team already exists`);
              } else {
                throw error;
              }
            }

            // Step 3: Add user to team
            try {
              await github.rest.teams.addOrUpdateMembershipForUserInOrg({
                org,
                team_slug: teamSlug,
                username,
                role: "member",
              });
              core.info(`✅ User ${username} added to team ${teamName}`);
            } catch (error) {
              core.warning(`⚠️ Could not add user: ${error.message}`);
            }

            // Step 4: Grant team access to repo
            try {
              await github.rest.teams.addOrUpdateRepoPermissionsInOrg({
                org,
                team_slug: teamSlug,
                owner: org,
                repo: repoName,
                permission: "push",
              });
              core.info(`✅ Team granted access to repo ${repoName}`);
            } catch (error) {
              core.warning(`⚠️ Failed to grant repo access: ${error.message}`);
            }

Required GitHub Secrets
Secret	Location	Description
ORG_ADMIN_TOKEN	Org or Repo secrets	PAT with admin:org, repo scopes
Usage Guide
Step 1: Access Backstage
javascript

Copy code
http://localhost:7007

Step 2: Navigate to Create
Click "Create" in the left sidebar or go to:

bash

Copy code
http://localhost:7007/create

Step 3: Select Template
Choose:

"Create Application Repo" - Simple repo creation
"Onboard User to Org" - Full onboarding workflow
Step 4: Fill the Form
Field	Example Value
GitHub Organization	my-org
GitHub Username	new-developer
Repository Name	my-new-service
Team Name	backend-team
Step 5: Click Create
Backstage will:

✅ Create the repository
✅ Add CODEOWNERS file
✅ Trigger team creation workflow
✅ Provide link to new repository
Troubleshooting
Issue: "Page Not Found" on /create
Cause: Scaffolder plugin not loaded.

Fix: Update packages/app/src/App.tsx:

typescript

Copy code
import { createApp } from '@backstage/frontend-defaults';
import catalogPlugin from '@backstage/plugin-catalog/alpha';
import scaffolderPlugin from '@backstage/plugin-scaffolder/alpha';
import { navModule } from './modules/nav';

export default createApp({
  features: [catalogPlugin, scaffolderPlugin, navModule],
});

Issue: "NotImplementedError: plugin.notifications.service"
Cause: Notifications plugin referenced but not installed.

Fix: Remove from packages/app/src/modules/nav/Sidebar.tsx:

typescript

Copy code
// Remove this line:
import { NotificationsSidebarItem } from '@backstage/plugin-notifications';

// And remove <NotificationsSidebarItem /> from JSX

Issue: GitHub Integration Fails
Check:

PAT token has correct scopes: repo, workflow, admin:org
Token is not expired
app-config.local.yaml is mounted correctly
Issue: Container Exits Immediately
Check logs:

bash

Copy code
docker logs <container_id>

Common causes:

Missing config file
Invalid YAML syntax
Port already in use
Team Handoff Guide
What Each Team Needs
Docker installed
GitHub PAT Token with scopes:
repo (Full control)
workflow (Actions)
admin:org (Team management)
platform-automation repository in their org with create-team.yml workflow
Setup Steps for New Teams
bash

Copy code
# Step 1: Pull the image
docker pull adeeashok/backstage:v1.2.0

# Step 2: Create local config
cat > app-config.local.yaml << 'EOF'
integrations:
  github:
    - host: github.com
      token: ghp_YOUR_PAT_TOKEN_HERE
EOF

# Step 3: Run the container
docker run -it -p 7007:7007 \
  -v $(pwd)/app-config.local.yaml:/app/app-config.local.yaml:ro \
  adeeashok/backstage:v1.2.0 \
  node packages/backend --config app-config.yaml --config app-config.local.yaml

# Step 4: Access UI
# Open http://localhost:7007/create

Docker Hub
Image: adeeashok/backstage

Tags:

Tag	Description
latest	Latest stable version
v1.2.0	Current release
Pull:

bash

Copy code
docker pull adeeashok/backstage:v1.2.0