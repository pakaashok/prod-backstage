🚀 Backstage Developer Portal - Containerized
A containerized Backstage developer portal with GitHub integration for automated repository creation, team management, and user onboarding.

📋 Table of Contents
Overview
Features
Architecture
Prerequisites
Technology Stack
Project Structure
Quick Start
Build from Source
Configuration Files
Templates
GitHub Integration
Usage Guide
Troubleshooting
Team Handoff Guide
Overview
This project provides a fully containerized Backstage developer portal that enables:

Automated GitHub repository creation with proper CODEOWNERS
Team creation and user onboarding via GitHub Actions
Self-service developer workflows through a web UI
Features
Feature	Description
🐳 Containerized	Runs in Docker for consistency across environments
🔐 Secure Secrets	PAT tokens stored in local config, not in image
🏢 Multi-Org Support	Works with any GitHub organization
📝 Auto CODEOWNERS	Automatically creates CODEOWNERS file in new repos
👥 Team Management	Creates teams and assigns users via GitHub Actions
🎨 Self-Service UI	User-friendly web interface for all operations
Architecture
Layer	Technology	Purpose
Host	Windows/Mac/Linux	Development machine
VM	Vagrant + Debian	Isolated environment
Container	Docker	Application runtime
Application	Backstage (Node.js 24)	Developer portal
Integration	GitHub API	Repository and team management
Workflow
User accesses Backstage UI at http://localhost:7007
User fills out template form (org, repo name, team, user)
Backstage creates GitHub repository with CODEOWNERS
Backstage triggers GitHub Actions workflow
Workflow creates team, invites user, grants permissions
Port Mapping
Layer	Port	URL
Windows Host	7007	http://localhost:7007
Vagrant VM	7007	http://192.168.56.10:7007
Docker Container	7007	Internal
Prerequisites
For Using Pre-built Image
Docker (20.10+)
GitHub Personal Access Token with scopes: repo, workflow, admin:org
For Building from Source
Vagrant (2.3+)
VirtualBox (7.0+)
Git Bash (Windows) or Terminal (Mac/Linux)
6GB+ RAM available for VM
GitHub Requirements
GitHub Organization
platform-automation repository with create-team.yml workflow
ORG_ADMIN_TOKEN secret configured
Technology Stack
Component	Technology	Version
Runtime	Node.js	24.x
Package Manager	Yarn	4.4.1
Framework	Backstage	Latest
Container	Docker	20.10+
VM	Vagrant + VirtualBox	2.3+ / 7.0+
Base Image	node:24-trixie-slim	-
Database	SQLite (in-memory)	-
OS (VM)	Debian Bookworm	12
Project Structure
css

Copy code
backstage/
├── Dockerfile
├── .dockerignore
├── app-config.yaml
├── app-config.local.yaml
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

Quick Start
Step 1: Pull Image
bash

Copy code
docker pull adeeashok/backstage:v1.2.0

Step 2: Create Local Config
Create app-config.local.yaml with your GitHub PAT:

yaml

Copy code
integrations:
  github:
    - host: github.com
      token: ghp_YOUR_PAT_TOKEN_HERE

Step 3: Run Container
bash

Copy code
docker run -it -p 7007:7007 \
  -v $(pwd)/app-config.local.yaml:/app/app-config.local.yaml:ro \
  adeeashok/backstage:v1.2.0 \
  node packages/backend --config app-config.yaml --config app-config.local.yaml

Step 4: Access UI
Open http://localhost:7007/create

Build from Source
Clone the repository
Start Vagrant VM: vagrant up && vagrant ssh
Navigate to backstage folder: cd ~/backstage
Build Docker image: docker build -t backstage .
Tag and push to registry:
bash

Copy code
docker tag backstage yourusername/backstage:v1.0.0
docker push yourusername/backstage:v1.0.0

Configuration Files
File	Purpose	Commit to Git?
app-config.yaml	Main Backstage configuration	Yes
app-config.local.yaml	GitHub PAT and secrets	No (gitignored)
Dockerfile	Multi-stage Docker build	Yes
.dockerignore	Docker build exclusions	Yes
Vagrantfile	VM configuration	Yes
Templates
Template 1: Create Application Repo
File: examples/template/template.yaml

Creates a GitHub repository with CODEOWNERS file.

Input	Description
GitHub Organization	Your GitHub org name
Repository Name	Name for the new repo
Team Name	Team to assign as code owners
Template 2: Onboard User to Org
File: examples/template/onboard-user-template.yaml

Full onboarding workflow - creates repo, team, and assigns user.

Input	Description
GitHub Organization	Your GitHub org name
GitHub Username	User to onboard
Repository Name	Name for the new repo
Team Name	Team to create and assign
GitHub Integration
Required Setup in Your GitHub Org
Create repository: platform-automation
Add workflow file: .github/workflows/create-team.yml
Add secret: ORG_ADMIN_TOKEN (PAT with admin:org, repo scopes)
Workflow Actions
The create-team.yml workflow performs:

Invites user to organization
Creates team (if not exists)
Adds user to team
Grants team push access to repository
Usage Guide
Open http://localhost:7007/create
Select a template (Create Application Repo or Onboard User to Org)
Fill in the form:
GitHub Organization
GitHub Username (for onboarding)
Repository Name
Team Name
Click Create
Follow the output link to your new repository
Troubleshooting
Issue	Cause	Solution
404 on /create	Scaffolder plugin missing	Add scaffolderPlugin to App.tsx
Notifications error	Plugin not installed	Remove NotificationsSidebarItem from Sidebar.tsx
GitHub auth fails	Invalid/expired token	Update token in app-config.local.yaml
Container exits	Config error	Check docker logs container_id
Port in use	Another service on 7007	Stop other service or change port
Team Handoff Guide
What Each Team Needs
Docker installed
GitHub PAT with scopes: repo, workflow, admin:org
platform-automation repo in their org with create-team.yml workflow
Setup Steps
bash

Copy code
docker pull adeeashok/backstage:v1.2.0

cat > app-config.local.yaml << 'EOF'
integrations:
  github:
    - host: github.com
      token: ghp_THEIR_TOKEN_HERE
EOF

docker run -it -p 7007:7007 \
  -v $(pwd)/app-config.local.yaml:/app/app-config.local.yaml:ro \
  adeeashok/backstage:v1.2.0 \
  node packages/backend --config app-config.yaml --config app-config.local.yaml

Docker Hub
Image	Tag
adeeashok/backstage	latest
adeeashok/backstage	v1.2.0
bash

Copy code
docker pull adeeashok/backstage:v1.2.0
