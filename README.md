# 🏨 Hotelier Infra - Local Development Environment

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=for-the-badge&logo=postgresql&logoColor=white)](https://www.postgresql.org/)

**Hotelier Infra** is the orchestration repository for the **Hotelier Suite**. It provides a single Docker Compose setup to run the entire local environment:

- PostgreSQL database
- pgAdmin UI
- Hotelier Backend API (NestJS)
- Hotelier Frontend (Next.js)

This repo is intentionally lightweight and focuses only on **local development & onboarding**.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Related Repositories](#-related-repositories)
- [Folder Layout](#-folder-layout)
- [Getting Started](#-getting-started)
- [Environment Configuration](#-environment-configuration)
- [Running the Stack](#-running-the-stack)
- [Available Services](#-available-services)
- [Useful Docker Commands](#-useful-docker-commands)
- [Future Evolution](#-future-evolution)
- [Contributing](#-contributing)
- [License](#-license)
- [Developer](#-developer)

---

## 🔍 Overview

This repository contains the **Docker Compose configuration** and shared **environment variables** required to spin up the full Hotelier Suite locally with a single command.

- Builds the **backend** and **frontend** directly from their sibling repositories
- Mounts the `src` directories for live code reload during development
- Provides a local PostgreSQL instance and pgAdmin for database management

> **Goal:** New developers should be able to clone three repos, copy one env file, and immediately run the entire stack.

---

## 🔗 Related Repositories

- **Backend API:** `hotelier-backend`
- **Frontend Application:** `hotelier-frontend`

All three repositories live under the same GitHub organization:

- `hotelier-suite/hotelier-backend`
- `hotelier-suite/hotelier-frontend`
- `hotelier-suite/hotelier-infra` _(this repo)_

---

## 🗂️ Folder Layout

For local development, the recommended directory structure is:

```bash
hotelier-suite/                # Just a folder on your machine (not a repo)
  hotelier-backend/           # git clone https://github.com/hotelier-suite/hotelier-backend.git
  hotelier-frontend/          # git clone https://github.com/hotelier-suite/hotelier-frontend.git
  hotelier-infra/             # git clone https://github.com/hotelier-suite/hotelier-infra.git
    compose.yaml
    .env.example
```

The `compose.yaml` in `hotelier-infra` assumes this **sibling layout** and uses relative paths like:

- `../hotelier-backend` as the backend build context
- `../hotelier-frontend` as the frontend build context

If you change the folder layout, you must update `compose.yaml` accordingly.

---

## 🚀 Getting Started

### 1. Prerequisites

- **Docker** and **Docker Compose v2** installed
- **Git** installed

### 2. Clone the repositories

From your workspace directory:

```bash
mkdir hotelier-suite
cd hotelier-suite

# Clone backend, frontend, and infra
git clone https://github.com/hotelier-suite/hotelier-backend.git
git clone https://github.com/hotelier-suite/hotelier-frontend.git
git clone https://github.com/hotelier-suite/hotelier-infra.git
```

### 3. Configure environment variables (standard flow)

Follow these steps to run the full stack using `hotelier-infra`:

1. Clone the three sibling repositories:
   - `hotelier-backend`
   - `hotelier-frontend`
   - `hotelier-infra`
2. In `hotelier-backend/`, create a Docker-specific env file based on the example:
   ```bash
   cp .env.docker.example .env.docker
   ```
3. In `hotelier-frontend/`, create a Docker-specific env file based on the example:
   ```bash
   cp .env.docker.example .env.docker
   ```
4. In `hotelier-infra/`, create the shared env file:
   ```bash
   cp .env.example .env
   ```
5. From `hotelier-infra/`, start the full stack:
   ```bash
   docker compose up --build
   ```

> **Important:** All `.env` / `.env.docker` files are **git-ignored** and should never be committed. Only the `*.example` templates are tracked in Git.

Docker will:

- Build the backend image from `../hotelier-backend`
- Build the frontend image from `../hotelier-frontend`
- Start PostgreSQL and pgAdmin
- Wire all containers together with the correct network and environment variables

To stop the stack:

```bash
docker compose down
```

To stop and remove volumes (including database data):

```bash
docker compose down -v
```

---

## ⚙️ Environment Configuration

The environment configuration is split between:

- **Infra-level wiring** (this repo): `hotelier-infra/.env` (based on `.env.example`)
- **Service-level settings** (each repo): `.env` / `.env.docker` files

### Infra-level env (`hotelier-infra/.env`)

This file only contains **shared wiring values** such as database credentials, ports, and public URLs:

```env
# PostgreSQL
POSTGRES_DB=hotelier_dev
POSTGRES_USER=hotelier_dev
POSTGRES_PASSWORD=hotelier_dev_password
POSTGRES_PORT=5432
DATABASE_URL=postgresql://hotelier_dev:hotelier_dev_password@postgres:5432/hotelier_dev

# pgAdmin
PGADMIN_EMAIL=admin@hotelier.dev
PGADMIN_PASSWORD=admin123

# Ports
API_PORT=3001
APP_PORT=3000

# Public URLs
FRONTEND_URL=http://localhost:3000
API_URL=http://localhost:3001/api
```

### Service-level envs

- **Backend (`hotelier-backend`)**
  - `.env.example` – template for running the backend standalone.
  - `.env.docker.example` – template for Docker/`hotelier-infra` usage.
  - `.env.docker` – local Docker env file (ignored by Git), referenced from `compose.yaml`.

- **Frontend (`hotelier-frontend`)**
  - `.env` / `.env.local` – for standalone Next.js development.
  - `.env.docker.example` – template for Docker/`hotelier-infra` usage.
  - `.env.docker` – local Docker env file (ignored by Git), referenced from `compose.yaml`.

`docker compose` loads the per-service `.env.docker` files and combines them with the wiring values from `hotelier-infra/.env`. Production secrets should live in a **secret manager** / CI and **never** in this repo.

---

## 🧩 Running the Stack

The `compose.yaml` file defines four main services:

- **postgres** – Core PostgreSQL database for the platform
- **pgadmin** – Web UI to inspect and manage the database
- **backend** – NestJS API from `hotelier-backend`
- **frontend** – Next.js app from `hotelier-frontend`

### Hot reload for backend & frontend

The compose file mounts the `src` folders from the sibling repos into the containers:

- `../hotelier-backend/src:/app/src`
- `../hotelier-frontend/src:/app/src`

This allows you to edit code in your local IDE while the containers are running and see changes without rebuilding the images.

---

## 🌐 Available Services

Once `docker compose up` is running, by default you get:

- **Frontend:** `http://localhost:3000`
- **Backend API:** `http://localhost:3001/api`
- **pgAdmin:** `http://localhost:5050`
  - Email: value from `PGADMIN_EMAIL`
  - Password: value from `PGADMIN_PASSWORD`
- **PostgreSQL:** `localhost:5432`
  - Database: `POSTGRES_DB`
  - User: `POSTGRES_USER`
  - Password: `POSTGRES_PASSWORD`

If you change ports in `.env`, the exposed URLs will adjust accordingly.

---

## 🧰 Useful Docker Commands

From inside `hotelier-infra/`:

```bash
# Start all services (build if needed)
docker compose up --build

# Start in detached mode
docker compose up -d

# View logs for all services
docker compose logs -f

# View logs for a single service
docker compose logs -f backend

# Stop services (keep volumes)
docker compose down

# Stop services and remove volumes (DB data)
docker compose down -v
```

If you run into strange issues, you can also prune unused Docker resources (be careful, this is global):

```bash
docker system prune -a
```

---

## 🧭 Future Evolution

As the Hotelier platform evolves into a **microservices architecture**, this repository can be extended to:

- Add additional services (auth-service, booking-service, notification-service, etc.)
- Provide multiple compose files (e.g. `compose.dev.yaml`, `compose.microservices.yaml`)
- Integrate with local message brokers, caches, and analytics stacks

The goal is to keep **all local orchestration** in one place so onboarding remains simple even as the system grows.

---

## 🤝 Contributing

This repository is part of the Hotelier Suite portfolio project and primarily serves as a **local development helper**. Suggestions for improving the developer experience, onboarding, or Docker setup are welcome.

---

## 📄 License

This project is open source and available for portfolio demonstration purposes.

---

## 👨‍💻 Developer

**Juan Nicolas Pardo Torres**

- LinkedIn: [nicolas-pardo-6156a1222](https://linkedin.com/in/nicolas-pardo-6156a1222)
- GitHub: [@Nick220505](https://github.com/Nick220505)
- Email: juannicolaspardo@gmail.com
