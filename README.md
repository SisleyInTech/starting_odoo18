# starting_odoo18

Docker-based **Odoo 18** development starter for building and running **in-house addons** under `custom/own/`. The container uses the official `odoo:18.0` image; optional source under `odoo_18_core/` is for local reference and IDE search only.

---

## Prerequisites

| Tool | Purpose |
|------|---------|
| **Git** | Clone this repository (and optionally the Odoo core) |
| **Docker** & **Docker Compose** | Run Odoo 18 in a container |
| **PostgreSQL** | Database reachable from the Odoo container (hostname set in `docker/.env`) |

PostgreSQL must be on the Docker network **`own_network`** (created by Compose or manually). The default host in `.env.example` is `postgres_db_17` — use the container name or alias that matches your stack.

---

## Quick start

### 1. Clone the repository

```bash
git clone <REPOSITORY_URL> starting_odoo18
cd starting_odoo18
```

### 2. Configure environment variables

```bash
cd docker
cp .env.example .env
```

Edit `docker/.env` with your PostgreSQL credentials and database name. **Do not commit** `.env`.

| Variable | Role |
|----------|------|
| `POSTGRES_*` | Connection to PostgreSQL |
| `POSTGRES_DB` | Odoo database name (when using `-d`) |
| `ADMIN_PASSWD` | Odoo **master password** (database manager: create, restore, drop, duplicate) |
| `EXEC_ODOO` | Extra Odoo CLI flags on container start (e.g. `-u own_my_module`) |

### 3. Start Odoo (development)

```bash
cd docker
docker compose -f docker-compose-dev.yml up -d
```

Follow logs if needed:

```bash
docker compose -f docker-compose-dev.yml logs -f own-odoo18-dev
```

### 4. Open Odoo

**http://localhost:8049/**

On first run you may see the **database manager**. When creating a database, enter the same value as `ADMIN_PASSWD` in `docker/.env` as the master password.

To install custom modules: enable **developer mode** → **Apps** → **Update Apps List** → install your module (e.g. `own_*` under `custom/own/`).

---

## (Recommended) Odoo 18 core source

Optional for running the stack, but very useful for development (browse models, views, and APIs in your IDE):

```bash
cd odoo_18_core
git clone --depth 1 --branch 18.0 https://github.com/odoo/odoo.git odoo18
cd ..
```

Omit `--depth 1` for a full Git history. Details: [`odoo_18_core/odoo_18_core.md`](odoo_18_core/odoo_18_core.md).

The dev container **does not mount** this folder; it runs Odoo from the Docker image.

---

## Repository layout

```
starting_odoo18/
├── config/
│   └── odoo-dev.conf       # Dev addons path, dev_mode, auto_reload
├── custom/
│   └── own/                # In-house modules (own_* prefix)
├── docker/
│   ├── docker-compose-dev.yml
│   ├── .env.example        # Template — copy to .env
│   └── .env                # Local secrets (gitignored)
├── docs/
│   └── local-development-guide.md
├── odoo_18_core/           # Optional Odoo source (not in Git)
└── README.md
```

**Rule:** customize business logic in `custom/own/`, not inside `odoo_18_core/`.

---

## Custom modules

| Topic | Location |
|-------|----------|
| Naming, checklist, manifest template | [`custom/own/own-module-guide.md`](custom/own/own-module-guide.md) |
| Addons path (dev) | `config/odoo-dev.conf` → `/mnt/extra-addons/custom/own` |

After changing Python, XML, data files, or `__manifest__.py`, upgrade the module:

- Set `-u <module_name>` in `EXEC_ODOO` in `docker/.env` and restart the container, or  
- **Apps** → your module → **Upgrade**

Example `EXEC_ODOO` (see `docker/.env.example` for more):

```env
EXEC_ODOO=--db_host=${POSTGRES_HOST} --db_port=${POSTGRES_PORT} --db_user=${POSTGRES_USER} --db_password=${POSTGRES_PASSWORD} -d ${POSTGRES_DB} -u own_my_module -c /var/lib/odoo/odoo.runtime.conf
```

---

## Useful commands

```bash
# From docker/
docker compose -f docker-compose-dev.yml up -d      # Start
docker compose -f docker-compose-dev.yml down     # Stop and remove container
docker compose -f docker-compose-dev.yml restart  # After .env changes
docker compose -f docker-compose-dev.yml logs -f    # Follow logs

# Shell inside the container
docker exec -it own-odoo18-dev bash
```

---

## Documentation

| Document | Description |
|----------|-------------|
| [Local development guide](docs/local-development-guide.md) | Full setup, workflow, module anatomy, troubleshooting |
| [In-house module guide](custom/own/own-module-guide.md) | Standards and checklist for `custom/own/` modules |
| [Odoo core folder](odoo_18_core/odoo_18_core.md) | How to clone and use `odoo_18_core/` |
| [Odoo 18 (official)](https://www.odoo.com/documentation/18.0/) | Framework and app documentation |

---

## One-line summary

**Configure `docker/.env` → start Compose on port 8049 → develop addons in `custom/own/` → use the guides above for conventions and day-to-day workflow.**
