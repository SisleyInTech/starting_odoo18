# Local Development Guide — Company name (Odoo 18)

A guide for developers **getting started** with this project: how to run the environment, understand the repository layout, and take your first steps building custom modules.

---

## Table of contents

1. [Prerequisites](#1-prerequisites)
2. [Repository structure](#2-repository-structure)
3. [Odoo core source code (`odoo_18_core/`)](#3-odoo-core-source-code-odoo_18_core)
4. [Custom modules (`custom/`)](#4-custom-modules-custom)
5. [Step-by-step setup](#5-step-by-step-setup)
6. [Key files explained](#6-key-files-explained)
7. [First access to Odoo](#7-first-access-to-odoo)
8. [Basic development workflow](#8-basic-development-workflow)
9. [Module anatomy (project example)](#9-module-anatomy-project-example)
10. [Useful commands](#10-useful-commands)
11. [Common issues](#11-common-issues)
12. [Related documentation](#12-related-documentation)

---

## 1. Prerequisites

Before you begin, make sure you have:

| Tool | Role in this project |
|------|----------------------|
| **Git** | Clone the repository and, optionally, the Odoo core |
| **Docker** and **Docker Compose** | Run Odoo 18 in a container |
| **Accessible PostgreSQL** | Database (usually another container on the `bn_network` network) |

Helpful background (not required on day one):

- Basic **Python** and **XML** (QWeb views in Odoo).
- **Docker** basics (containers, networks, volumes).
- Familiarity with the [official Odoo 18 documentation](https://www.odoo.com/documentation/18.0/).

---

## 2. Repository structure

Simplified view of the folders you will use most:

```
starting_odoo18/
├── config/                 # Odoo configuration (odoo.conf, odoo_prod.conf)
├── custom/                 # Third-party and Company name custom modules
│   └── own/               # Namespace for in-house modules (Company name)
│       └── own_name_module/    # Example: website customizations
├── docker/                 # Docker Compose and environment variables (.env)
├── docs/                   # Project documentation (this guide, translations, etc.)
├── odoo_18_core/           # Odoo 18 source code (not versioned in Git)
└── README.md               # Quick project overview
```

**Key idea:** Git tracks configuration and modules under `custom/`. You download the **Odoo core** into `odoo_18_core/` for local reference; the development container uses the official `odoo:18.0` image.

---

## 3. Odoo core source code (`odoo_18_core/`)

The `odoo_18_core/` folder is reserved for the **official Odoo 18 source code**. It is not part of this project’s Git history (only the instructions file `odoo_18_core.md` is tracked).

### Why keep it if Docker already includes Odoo?

- **Read and understand** how the framework works (models, controllers, standard views).
- **Search in your IDE** for core classes, methods, and examples.
- **Debug** or compare behavior against the real version 18 codebase.

The Docker environment **does not mount** this folder by default: it runs Odoo from the `odoo:18.0` image. Your copy under `odoo_18_core/` is a local reference for developers.

### How to obtain the core

From the repository root:

```bash
cd odoo_18_core
git clone --depth 1 --branch 18.0 https://github.com/odoo/odoo.git odoo18
cd ..
```

You may use a different subdirectory name (e.g. `odoo` instead of `odoo18`); what matters is that **all Odoo 18 code stays under `odoo_18_core/`** and is not mixed with your custom modules.

More detail: [`odoo_18_core/odoo_18_core.md`](../odoo_18_core/odoo_18_core.md).

---

## 4. Custom modules (`custom/`)

Everything under `custom/` is **addons that are not the Odoo core**:

| Type | Description | Example in this repo |
|------|-------------|----------------------|
| **In-house modules** | Built by Company name | `custom/own/own_name_module/` |
| **Third-party modules** | OCA, partners, purchased apps, etc. | Install here when the team adds them |

### Folder conventions

- **`custom/own/`** — prefix/namespace for internal modules (`own` = Company name).
- Each module is a folder with `__manifest__.py` (e.g. `own_name_module`).

In development, `config/odoo.conf` sets the addons path:

```ini
addons_path = /mnt/extra-addons/custom/own
```

On the host, that maps to `starting_odoo18/custom/own`. If you later add third-party modules elsewhere (e.g. `custom/third_party`), **extend** `addons_path` in `odoo.conf` with comma-separated paths, for example:

```ini
addons_path = /mnt/extra-addons/custom/own,/mnt/extra-addons/custom/third_party
```

(and mount those folders in Docker if needed).

**Practical rule:** do not change code inside `odoo_18_core/` to customize the business; create or extend modules in `custom/`.

---

## 5. Step-by-step setup

### 5.1 Clone the repository

```bash
git clone <REPOSITORY_URL> starting_odoo18
cd starting_odoo18
```

### 5.2 (Recommended) Download Odoo 18 core

See [section 3](#3-odoo-core-source-code-odoo_18_core). Optional to **start** the container, but very useful for **development**.

### 5.3 Docker network and PostgreSQL

The development compose file attaches Odoo to the external network **`bn_network`**. The Odoo container expects PostgreSQL reachable at the hostname you set in `.env` (default in the example: `postgres_db_17`).

If the network does not exist:

```bash
docker network create bn_network
```

PostgreSQL must be on the **same network** and listen on the configured port (usually `5432`). The service/host name (`POSTGRES_HOST`) must match the container name or DNS alias in Docker.

> If PostgreSQL is not running yet, coordinate with the team on the `docker-compose` or stack that exposes `postgres_db_17` on `bn_network`.

### 5.4 Environment variables

Copy the example and edit it with real values:

```bash
cd docker
cp .env.example .env
```

Edit `docker/.env` (this file is **not** committed to Git). Main variables:

| Variable | Meaning |
|----------|---------|
| `POSTGRES_HOST` | PostgreSQL hostname in Docker |
| `POSTGRES_PORT` | Port (typically `5432`) |
| `POSTGRES_USER` / `POSTGRES_PASSWORD` | Database credentials |
| `POSTGRES_DB` | Odoo database name |
| `ADMIN_PASSWD` | Odoo **master password** (database manager: create, duplicate, restore) |
| `EXEC_ODOO` | Arguments used when starting `odoo` in the container |

### 5.5 Start Odoo for development

From the `docker/` folder (so Compose loads `.env`):

```bash
docker compose -f docker-compose-dev.yml up -d
```

View logs:

```bash
docker compose -f docker-compose-dev.yml logs -f xbn-odoo-dev
```

### 5.6 Open the application

Browser: **http://localhost:8062/**

Host port **8062** maps to container port **8069** (Odoo).

---

## 6. Key files explained

### 6.1 `docker/docker-compose-dev.yml`

Defines the **`xbn-odoo-dev`** service:

- **Image:** `odoo:18.0`
- **Port:** `8062:8069`
- **Environment:** read from `docker/.env` (Postgres, `ADMIN_PASSWD`, etc.)
- **Volumes:**
  - `odoo_xbn_data` → persistent Odoo data (`/var/lib/odoo`)
  - `../custom` → `/mnt/extra-addons/custom` (your modules)
  - `../config` → `/etc/odoo` (`.conf` files)
- **Network:** `bn_network` (fixed name, external network)
- **Entrypoint:** waits for Postgres (`pg_isready`), copies `odoo.conf` to a runtime file, injects `admin_passwd` from `ADMIN_PASSWD`, and runs `odoo` with `${EXEC_ODOO}`

This keeps the master password out of the committed `.conf`: it is added at startup from the environment.

### 6.2 `config/odoo.conf`

Configuration used in **development** (mounted in the container):

| Option | Value | Meaning |
|--------|--------|---------|
| `addons_path` | `/mnt/extra-addons/custom/own` | Where Odoo looks for custom modules |
| `data_dir` | `/var/lib/odoo` | Session files, generated assets, etc. |
| `list_db` | `True` | Web database selector (create/choose databases) |
| `dev_mode` | `True` | Development mode (more tools and reloads) |
| `auto_reload` | `True` | Auto-reload on Python/XML changes (useful in dev) |
| `log_level` | `info` | Log level |
| `debug_mode` | `True` | Extra debug information |

**Production** uses `config/odoo_prod.conf` with `docker-compose-prod.yml`.

### 6.3 `docker/.env.example`

Documented template for variables. The most important day-to-day setting is **`EXEC_ODOO`**: it controls how Odoo starts each time you bring up the container.

Examples (replace credentials and database name with yours):

**Start with a specific database only:**

```env
EXEC_ODOO=--db_host=${POSTGRES_HOST} --db_port=${POSTGRES_PORT} --db_user=${POSTGRES_USER} --db_password=${POSTGRES_PASSWORD} -d ${POSTGRES_DB} -c /var/lib/odoo/odoo.runtime.conf
```

**Update a module on startup** (common after changing XML, Python, or `__manifest__.py`):

```env
EXEC_ODOO=--db_host=${POSTGRES_HOST} --db_port=${POSTGRES_PORT} --db_user=${POSTGRES_USER} --db_password=${POSTGRES_PASSWORD} -d ${POSTGRES_DB} -u own_name_module -c /var/lib/odoo/odoo.runtime.conf
```

**Update translations:**

```env
EXEC_ODOO=--db_host=${POSTGRES_HOST} --db_port=${POSTGRES_PORT} --db_user=${POSTGRES_USER} --db_password=${POSTGRES_PASSWORD} -d ${POSTGRES_DB} --i18n-overwrite -u all --stop-after-init -c /var/lib/odoo/odoo.runtime.conf
```

After changing `EXEC_ODOO`, restart the container:

```bash
docker compose -f docker-compose-dev.yml restart xbn-odoo-dev
```

---

## 7. First access to Odoo

1. Open **http://localhost:8062/**.
2. If no database exists, you will see the **database manager**.
3. When **creating** a new database, Odoo asks for the **master password**: use the same value as `ADMIN_PASSWD` in `docker/.env`.
4. Complete the setup wizard (language, country, demo data if applicable).
5. Log in with the admin user you define in the wizard.

To **install custom modules**:

1. Enable **developer mode** (Settings → activate developer mode, or `?debug=1` in the URL depending on version).
2. Apps → update apps list.
3. Find the module (e.g. **Company name Website** / `own_name_module`) and install it.

If `EXEC_ODOO` includes `-u own_name_module`, the module may update automatically on every container start.

---

## 8. Basic development workflow

### Typical cycle

1. Edit code in `custom/own/<your_module>/` (Python, XML, SCSS, etc.).
2. With `auto_reload` and `dev_mode`, many Python changes reload automatically; changes to **views**, **data**, or **`__manifest__.py`** usually require **updating the module** (`-u module_name`).
3. Ways to update:
   - Set `-u own_name_module` in `EXEC_ODOO` and restart the container, or
   - From the UI: Apps → module → Upgrade, or
   - Command line inside the container (see [useful commands](#10-useful-commands)).

### Project best practices

- UI and website copy in **English** in code; translations in `i18n/*.po`. See [`docs/translations.md`](translations.md).
- Do not commit `docker/.env` or secrets.
- Small, focused changes and descriptive commits (per team norms).
- Test locally before deploying with `docker-compose-prod.yml`.

### Debugging

- Check logs: `docker compose -f docker-compose-dev.yml logs -f xbn-odoo-dev`
- Odoo developer mode for technical menus, views, assets, etc.
- Browse the core under `odoo_18_core/` when you need to see how standard Odoo does it.

---

## 9. Module anatomy (project example)

The `own_name_module` module shows a typical layout:

```
custom/own/own_name_module/
├── __init__.py              # Imports subpackages (e.g. controllers)
├── __manifest__.py          # Metadata, dependencies, data files, assets
├── controllers/             # HTTP routes (optional)
├── views/                   # QWeb templates and XML views
├── static/src/              # SCSS, images, frontend JS
└── i18n/                    # Translations (.po)
```

Relevant parts of `__manifest__.py`:

- **`depends`:** modules that must be installed first (`website`, `website_crm`, …).
- **`data`:** XML files loaded on install/upgrade.
- **`assets`:** frontend bundles (e.g. SCSS on `web.assets_frontend`).

When creating a **new module**:

1. Folder under `custom/own/my_module/`.
2. `__manifest__.py` with `version` like `18.0.x.y.z`.
3. `__init__.py` (even if empty or only importing controllers/models).
4. Refresh the apps list and install from Odoo, or use `-i my_module` in `EXEC_ODOO` the first time.

Official docs: [Module tutorials](https://www.odoo.com/documentation/18.0/developer/tutorials.html).

---

## 10. Useful commands

```bash
# From docker/
docker compose -f docker-compose-dev.yml up -d          # Start
docker compose -f docker-compose-dev.yml down           # Stop and remove container
docker compose -f docker-compose-dev.yml restart        # Restart after .env changes
docker compose -f docker-compose-dev.yml logs -f        # Follow logs

# Shell inside the Odoo container
docker exec -it xbn-odoo-dev bash

# Example: upgrade a module manually (inside the container)
odoo -d YOUR_DB -u own_name_module --stop-after-init \
  -c /var/lib/odoo/odoo.runtime.conf \
  --db_host=postgres_db_17 --db_user=... --db_password=...
```

Replace `YOUR_DB` and credentials with your `.env` values.

---

## 11. Common issues

| Symptom | Likely cause | What to do |
|---------|--------------|------------|
| Container restarts in a loop | Postgres unreachable | Ensure `postgres_db_17` (or your host) is on `bn_network` and `pg_isready` works |
| `network bn_network not found` | Network not created | `docker network create bn_network` |
| Custom modules not listed | `addons_path` or volume | Check `config/odoo.conf` and that `custom` is mounted in compose |
| XML changes not visible | Module not upgraded | `-u module_name` or Upgrade from Apps |
| Master password rejected | Mismatch | Use exactly `ADMIN_PASSWD` from `.env` when creating/restoring a DB |
| Port 8062 in use | Another service | Change the mapping in `docker-compose-dev.yml` or free the port |

---

## 12. Related documentation

| Resource | Location |
|----------|----------|
| Quick repo overview | [`README.md`](../README.md) |
| Local Odoo core | [`odoo_18_core/odoo_18_core.md`](../odoo_18_core/odoo_18_core.md) |
| Translations (i18n) | [`docs/translations.md`](translations.md) |
| Brand manual | [`docs/brand/brand_manual.md`](brand/brand_manual.md) |
| Example web module | [`custom/own/own_name_module/README.md`](../custom/own/own_name_module/README.md) |
| Odoo 18 (official) | https://www.odoo.com/documentation/18.0/ |

---

## One-line summary

**Clone the repo → configure `docker/.env` and `bn_network` with PostgreSQL → optionally clone Odoo into `odoo_18_core/` for core reference → develop in `custom/own/` → run Docker on port 8062 and upgrade modules with `-u` or from the UI.**

---

*Company name — internal onboarding document. Property of Company name. Coordinate with the team for shared Postgres credentials or stack in your environment.*
