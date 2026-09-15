# WordPress Docker Setup

A containerized WordPress environment orchestrated with Docker Compose,
consisting of a WordPress service with Apache and a MySQL database service.

## Table of Contents

- [Description](#description)
- [Repository Contents](#repository-contents)
- [Quickstart](#quickstart)
  - [Prerequisites](#prerequisites)
  - [Setup](#setup)
- [Usage](#usage)
  - [Environment Variables](#environment-variables)
  - [Persistence](#persistence)
  - [Networking](#networking)
  - [Restart Behaviour](#restart-behaviour)
  - [Customization](#customization)
- [Troubleshooting](#troubleshooting)

## Description

This repository provides a reproducible WordPress installation for local
development and demonstration purposes. It defines two services:

| Service | Image | Role |
| --- | --- | --- |
| `wordpress` | `wordpress:php8.2-apache` | Application server, exposed to the host |
| `db` | `mysql:8.4` | Database backend, reachable only inside the stack |

Both services share a dedicated Docker network and store their state in named
volumes. Credentials are never committed to this repository; they are supplied
at runtime through an `.env` file.

## Repository Contents

| File | Description |
| --- | --- |
| `docker-compose.yaml` | Service, network and volume definitions for the stack |
| `example.env` | Template for the required environment variables. Copy to `.env` and fill in your own values |
| `.gitignore` | Excludes the `.env` file and OS-specific artefacts from version control |
| `Wordpress Checkliste.pdf` | Project requirements provided by the Developer Akademie. Reference material, not part of the application |

## Quickstart

### Prerequisites

- Docker Compose v2

Verify your installation with:

```bash
docker compose version
```

### Setup

```bash
# 1. Clone the repository
git clone <repository-url>
cd dso-wordpress

# 2. Create your environment file from the template
cp example.env .env

# 3. Edit .env and set your own database credentials
#    This file is ignored by git and must never be committed.

# 4. Start the stack
docker compose up -d
```

WordPress is then available at `http://<host>:8080`, where `<host>` is
`localhost` for a local setup or the address of your server when deployed
remotely. On first access, WordPress guides you through its installation
wizard.

## Usage

### Environment Variables

The stack distinguishes between values that are safe to publish and values that
are not. Non-sensitive settings are defined directly in `docker-compose.yaml`;
credentials are read from `.env`, which is excluded via `.gitignore`.

| Variable | Description | Default | Required |
| --- | --- | --- | --- |
| `WP_PORT` | Host port WordPress is published on | `8080` | no |
| `DB_USER` | Database user created on first startup | – | **yes** |
| `DB_PASSWORD` | Password for that user | – | **yes** |

`DB_USER` and `DB_PASSWORD` are declared as required variables in
`docker-compose.yaml`:

```yaml
MYSQL_USER: ${DB_USER:?DB_USER is required in .env}
```

If either is missing **or empty**, Compose aborts with an explicit error
instead of starting the stack in a broken state. Credentials deliberately have
no default value: a default password is a backdoor, not a convenience.

The MySQL root account is created with `MYSQL_RANDOM_ROOT_PASSWORD`, so a
random password is generated on first startup and never stored anywhere. The
application connects as `DB_USER`, which only has access to the `wordpress`
database — the root account is not needed during normal operation.

### Persistence

Both services store their state in named Docker volumes, which exist
independently of the containers:

| Volume | Mounted at | Contains |
| --- | --- | --- |
| `db` | `/var/lib/mysql` | Database contents: posts, pages, users and settings |
| `wordpress` | `/var/www/html` | WordPress core, themes, plugins and media uploads |

Because this data lives in volumes rather than in the containers' writable
layer, it survives recreating the containers:

```bash
docker compose down     # containers are removed, volumes are kept
docker compose up -d    # all data is still there
```

> **Warning:** `docker compose down -v` additionally removes the volumes. This
> permanently deletes the database and all uploaded media.

### Networking

Both services are attached to a single user-defined bridge network, `wp_net`.
Within that network, Docker provides DNS resolution by service name, which is
why WordPress addresses the database as `db` rather than by IP address:

```yaml
WORDPRESS_DB_HOST: db
```

This keeps the configuration portable and avoids hardcoding infrastructure
details into the repository.

Only the `wordpress` service publishes a port to the host (`8080` by default).
The database has **no** port mapping and is therefore not reachable from
outside the stack — it is only accessible to containers on `wp_net`.

### Restart Behaviour

Both services use `restart: on-failure`. If a container's main process
terminates with a non-zero exit code, Docker restarts it automatically.

Note that this policy reacts to the *exit code*, not to the container stopping
as such. A container shut down cleanly, or stopped manually via
`docker compose stop`, is not restarted.

### Customization

The setup can be adapted without editing `docker-compose.yaml`:

**Changing the host port** — set `WP_PORT` in your `.env` file, for example to
run WordPress on port 9000:

```
WP_PORT=9000
```

Only the host side of the mapping changes; the container continues to serve on
port 80 internally.

**Changing image versions** — image tags in `docker-compose.yaml` are pinned to
specific versions to keep the setup reproducible and identical across
environments. When upgrading, check that the PHP version still receives
security updates at https://www.php.net/supported-versions.php.

**Changing credentials after the first start** — MySQL creates users only when
the data directory is initialised, so editing `.env` afterwards has no effect
on the existing database. To apply new credentials, the database volume has to
be removed first:

```bash
docker compose down -v
docker compose up -d
```

This deletes all existing content.

## Troubleshooting

**`required variable DB_USER is missing a value`**
The `.env` file is missing or a variable is empty. Copy `example.env` to `.env`
and fill in all values.

**`Error establishing a database connection`**
Usually a credential mismatch: the values in `.env` differ from those the
database was initialised with. See *Changing credentials after the first start*
above.

**Port is already allocated**
Another process is using the host port. Set a different `WP_PORT` in `.env`.
