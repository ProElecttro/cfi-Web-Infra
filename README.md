# CFI Web Infrastructure

## Overview

The CFI Web Infrastructure is a centralized hosting and deployment platform for the **Centre for Innovation (CFI), IIT Madras**, supporting **14+ clubs and 60+ projects**.

The main goal was to remove the deployment bottleneck at the IITM Computer Center. Instead of every project team independently requesting server access, deployment, configuration, and maintenance, projects could use a shared infrastructure while remaining independently deployable.

The platform supports both the **public-facing CFI website** and the backend/CMS infrastructure used by clubs and projects.

## Architecture

```text
                    Users / Club Owners
                           |
                           v
                    CFI IITM Domain
                           |
                           v
                    Nginx Reverse Proxy
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
        Project A      Project B      Project C
        Container      Container      Container
             |             |             |
             +-------------+-------------+
                           |
                    IITM Infrastructure
                      Bare-metal Server
```

Each project is deployed independently in its own container. Nginx acts as the entry point and routes incoming requests to the appropriate project.

## Infrastructure

* Hosted on **IIT Madras infrastructure**
* Bare-metal server connected through the **IITM network**
* IITM **ERNET/network infrastructure**
* IITM-managed **DNS records**
* **Linux** server environment
* **Docker** for application isolation
* **Nginx** as reverse proxy
* **PM2** for Node.js process management and recovery
* **GitHub Actions** for automated builds/deployments
* Self-hosted GitHub Actions runner

## Project Isolation

With 60+ projects, allowing every project to directly manage the server would create operational and security problems.

Instead:

```text
CFI Server
│
├── Project A → Docker Container
├── Project B → Docker Container
├── Project C → Docker Container
├── Project D → Docker Container
└── ...
    └── Project 60+
```

Each project has its own application environment and can be built, restarted, and updated independently.

A failure in one project does not require restarting the entire CFI platform.

## Deployment Flow

A typical deployment follows this flow:

```text
Developer
    |
    v
GitHub Repository
    |
    v
GitHub Actions
    |
    v
Self-hosted Runner
    |
    v
Docker Build
    |
    v
Project Container
    |
    v
Nginx
    |
    v
Users
```

When a project team pushes changes to its repository, the corresponding GitHub Actions workflow is triggered.

The self-hosted runner executes the build and deployment workflow. The application is built into a Docker image and deployed as the project's container.

Multiple builds can be triggered at the same time. GitHub Actions manages the workflow jobs, while the self-hosted runner executes them according to available runner capacity.

## Request Routing

Projects may internally listen on different ports, commonly using ports such as `3000`.

Nginx provides a stable external entry point and handles the mapping between the incoming request and the appropriate project.

```text
user request
     |
     v
   Nginx
     |
     +----> Project A :3000
     |
     +----> Project B :3001
     |
     +----> Project C :3002
```

This means projects do not need to expose their internal ports directly to users.

## CMS and Club Access

The public CFI website primarily provides information such as:

* Clubs
* Projects
* Blogs
* Events
* Announcements
* Schedules

Behind the public interface is a CMS that allows authorized club/project owners to manage their content.

For example:

```text
Club Owner
    |
    v
CMS
    |
    v
Backend
    |
    v
Database
    |
    v
Public CFI Website
```

A club owner can therefore update content without requiring direct server access.

## Process Management

PM2 is used for Node.js process management.

It provides:

* Process monitoring
* Automatic restart after crashes
* Application status monitoring
* Log management
* Service recovery

Server health monitoring was also configured to notify administrators when services failed.

## Resource Management

Since many projects share the same physical infrastructure, resource contention can occur.

For example:

```text
Multiple builds
      |
      v
CPU usage increases
      |
      v
Server resource pressure
      |
      v
Swap provides additional virtual memory
```

Swap space was configured as a safety mechanism during periods of high memory pressure, particularly when multiple workloads were running on the server.

## Why This Architecture?

The infrastructure solves the main operational problem:

```text
60+ independent projects
          |
          v
Individual Computer Center deployment
          |
          v
Deployment bottleneck
```

becomes:

```text
60+ projects
     |
     v
Centralized CFI infrastructure
     |
     +---- Docker isolation
     +---- GitHub Actions
     +---- Self-hosted runner
     +---- Nginx routing
     +---- PM2 process management
     |
     v
Independent project deployments
```

The important idea is that **the infrastructure is centralized, while development and deployment remain decentralized for individual project teams**.
