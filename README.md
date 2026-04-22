# GSC 2026 - SwarajDesk

A full-stack grievance management platform built as a Bun monorepo.

It includes citizen and admin frontends, multiple backend services, AI-assisted processing, blockchain hash synchronization, monitoring, and infrastructure automation for cloud deployment.

## Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Services](#services)
- [How It Works](#how-it-works)
- [AI Workflow](#ai-workflow)
- [Tech Stack](#tech-stack)
- [Quick Start](#quick-start)
- [Testing](#testing)
- [Deployment and Infra](#deployment-and-infra)
- [License](#license)

## Overview

SwarajDesk is organized as a monorepo where each major capability is isolated into a package:

- Public grievance submission and user workflows
- Admin dashboard and moderation operations
- Complaint queue and background processing
- AI agents for automation and assistance
- Blockchain-backed integrity layer
- Monitoring and alerting
- Infrastructure as code and automation scripts

## Repository Structure

```text
GSC-2026-Primeagen/
├── packages/
│   ├── user-fe          # Citizen-facing frontend (Next.js)
│   ├── admin-fe         # Admin frontend (Next.js)
│   ├── user-be          # User backend API + WS support
│   ├── admin-be         # Admin backend API
│   ├── compQueue        # Complaint queue microservice
│   ├── agents           # AI agents service
│   ├── self             # Self-service backend
│   ├── blockchain-be    # Blockchain worker + contracts
│   ├── monitoring       # Health-check and alerting service
│   ├── k8s              # Kubernetes manifests/configs
│   └── mychart          # Helm chart assets
├── ansible/             # Automation playbooks
├── aws/                 # AWS-related IaC assets
├── ec2/                 # EC2 automation/deployment files
├── terraform/           # Terraform configuration
├── MDs/                 # Supporting project documents
└── GSC-dep.md           # Detailed deployment guide
```

## Services

Core application and platform services in this repo:

- `user-fe`: End-user web application
- `admin-fe`: Administrative dashboard
- `user-be`: Main user API and websocket-enabled backend
- `admin-be`: Admin-side API and business logic
- `compQueue`: Complaint processing queue microservice
- `agents`: AI orchestration service for task automation
- `self`: Self-service API
- `blockchain-be`: Blockchain worker and sync backend
- `monitoring`: Standalone service health and alerting tool

## How It Works

High-level request lifecycle:

1. User submits a complaint from `user-fe`
2. `user-be` validates/authenticates and pushes complaint payload into Redis registration queue
3. `compQueue` polls queue, validates, runs moderation, checks duplicates, writes complaint to DB
4. Processed complaint is forwarded for downstream assignment and blockchain sync workflows
5. Admin users work the complaint via `admin-fe` + `admin-be`
6. Status, reports, and insights are surfaced through app APIs and AI endpoints

## AI Workflow

SwarajDesk uses a multi-agent pattern in `packages/agents` with specialized flows:

1. **Sentient AI (primary assistant)**
- Handles user chat and guidance
- Uses tool-based actions for complaint help, status lookups, categories, trends, and navigation hints
- Can trigger complaint draft flow and location detection

2. **Help AI (escalation assistant)**
- Activated when Sentient AI marks a support escalation path
- Focuses on support workflows, knowledge search, profile-aware assistance, and human escalation tools

3. **Dedup AI (pre-submission duplicate detection)**
- Called via `/api/dedup`
- Checks similar complaints before final submission
- Returns `hasSimilar`, `isDuplicate`, candidate matches, and suggestion text

4. **Abuse AI (service-to-service moderation)**
- Exposed via `/api/moderate` (internal API key auth)
- Used by `compQueue` during processing
- Detects abusive content, masks flagged phrases, and returns moderation metadata

5. **Vision AI (image analysis and image matching)**
- `/api/image` analyzes uploaded evidence and suggests category/subcategory + complaint draft context
- `/api/match` compares two images for likely match confidence

This architecture keeps user conversation, moderation, deduplication, and visual intelligence isolated but composable.

## Tech Stack

- Runtime and workspace: Bun workspaces
- Frontend: Next.js, React, TailwindCSS
- Backend: Node/Bun services with Express/Hono
- Database layer: Prisma + PostgreSQL adapters
- Queue/cache/messaging: Redis clients
- AI integrations: OpenAI, Vertex AI, LangChain ecosystem
- Blockchain: Hardhat + ethers
- Infra/ops: Docker, Ansible, Terraform, Kubernetes, Helm

## Quick Start

### 1) Prerequisites

- Bun installed
- Node.js (for some package tooling)
- PostgreSQL and Redis (for local backend execution)

### 2) Install dependencies

Run at repository root:

```bash
bun install
```

### 3) Start services locally

Examples:

```bash
# Frontends
cd packages/user-fe && bun run dev
cd packages/admin-fe && bun run dev

# Core backends
cd packages/user-be && bun run dev
cd packages/admin-be && bun run dev
cd packages/compQueue && bun run dev

# AI and support services
cd packages/agents && bun run dev
cd packages/self && bun run dev
cd packages/monitoring && bun run dev

# Blockchain package
cd packages/blockchain-be && bun run dev:all
```

Note: service-specific environment files are required for full local startup.

## Testing

From repository root:

```bash
bun run test
bun run test:coverage
```

These aggregate tests for:

- `admin-be`
- `compQueue`
- `user-be`

Additional package-level tests are available via each package's `package.json`.

## Deployment and Infra

Detailed deployment steps are documented in:

- `GSC-dep.md` (VM + Docker Compose + Nginx + Cloudflare flow)
- `ansible/` for provisioning and automation
- `terraform/` and `aws/` for infrastructure provisioning
- `packages/k8s` and `packages/mychart` for Kubernetes/Helm resources

## License

This project is licensed under the terms of the repository [LICENSE](./LICENSE).
