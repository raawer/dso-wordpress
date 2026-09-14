# WordPress Docker Setup

A containerized WordPress environment orchestrated with Docker Compose,
consisting of a WordPress service and a MySQL database service.

## Table of Contents

- [Description](#description)
- [Repository Contents](#repository-contents)
- [Quickstart](#quickstart)
- [Usage](#usage)
  - [Environment Variables](#environment-variables)
  - [Persistence](#persistence)
  - [Networking](#networking)
  - [Customization](#customization)
- [Troubleshooting](#troubleshooting)

## Description

<!-- TODO: What is this repository for? Cover:
     - purpose of the project
     - which services are defined and how they interact
     - who this setup is intended for (local development / demo)
-->

## Repository Contents

| File | Description |
| --- | --- |
| `docker-compose.yaml` | <!-- TODO --> |
| `example.env` | Template for the required environment variables. Copy to `.env` and fill in your own values. |
| `.gitignore` | Excludes secrets (`.env`) and OS-specific files from version control. |
| `Wordpress Checkliste.pdf` | Project requirements provided by the Developer Akademie. Not part of the application itself. |

## Quickstart

### Prerequisites

<!-- TODO: list required software and minimum versions, e.g. Docker Engine, Docker Compose plugin -->

### Setup

```bash
# 1. Clone the repository
git clone <repository-url>
cd <repository-directory>

# 2. Create your environment file from the template
cp example.env .env

# 3. Edit .env and set your own credentials
#    (never commit this file)

# 4. Start the stack
docker compose up -d
```

WordPress is then available at `http://<host>:8080`.

## Usage

### Environment Variables

<!-- TODO: document every variable from example.env in this table.
     Note which are safe defaults and which MUST be changed. -->

| Variable | Description | Default |
| --- | --- | --- |
|  |  |  |

> **Security note:** Credentials are never stored in `docker-compose.yaml`
> or committed to this repository. They are supplied at runtime via `.env`,
> which is excluded through `.gitignore`.

### Persistence

<!-- TODO: explain which volume stores what, and why data survives
     `docker compose down` but not `docker compose down -v` -->

### Networking

<!-- TODO: explain how the two services reach each other and which
     ports are exposed to the host -->

### Customization

<!-- TODO: explain how to change the setup to get different results, e.g.
     - changing the published port
     - switching the database image or version
     - adjusting the restart policy
-->

## Troubleshooting

<!-- TODO: optional, but nice to have. Common issues and how to resolve them. -->
