# In-House Module Guide — Odoo 18

Reference document for **creating and maintaining internal modules** under `custom/own/`. Fill it in with your company details and use it as a checklist for every new module.

**Related:** local environment and development workflow → [`docs/local-development-guide.md`](../../docs/local-development-guide.md).

---

## 1. Company information (fill in once)

| Field | Your company value |
|-------|---------------------|
| **Legal / trade name** | _e.g. My Company Inc._ |
| **Short name (UI, docs)** | _e.g. MyCompany_ |
| **Technical module prefix** | `own` _(fixed in this repo: `custom/own/` folder)_ |
| **Author in `__manifest__.py`** | _e.g. My Company / IT Team_ |
| **Website / support** | _URL or internal email_ |
| **Default license** | _e.g. LGPL-3, OPL-1, proprietary — agreed with legal_ |
| **Source language for code** | **English** (strings in Python/XML; translations in `i18n/`) |
| **Published languages** | _e.g. en_US, es_ES, fr_FR_ |
| **Repository / main branch** | _Git URL, branch naming convention_ |
| **Technical contact** | _Person or channel (Slack, email)_ |

---

## 2. Naming conventions

### 2.1 Module name (folder and technical name in Apps)

| Rule | Good example | Avoid |
|------|--------------|-------|
| Folder under `custom/own/` | `own_crm_extra` | `crm_extra` (no prefix) |
| Lowercase letters, digits, and `_` only | `own_sale_discount` | `own-Sale`, spaces |
| Name = folder = technical name in Apps | `own_website_brand` | generic names (`utils`, `fix`) |
| Descriptive and scoped | `own_partner_fiscal_id` | `own_stuff` |

### 2.2 Display name (`name` in manifest)

- Suggested format: **`[Company short name] — Short description`**
- Example: `MyCompany — Website branding`

### 2.3 Version (`version` in manifest)

- Project format: **`18.0.<major>.<minor>.<patch>`**
- Examples: `18.0.1.0.0` (first release), `18.0.1.0.1` (fix), `18.0.2.0.0` (meaningful functional change)
- The leading segment is always the **Odoo version** (18).

### 2.4 Models, fields, and XML IDs

| Element | Convention |
|---------|------------|
| New model | `own.<resource>` — _e.g. `own.project.tag`_ |
| Inheritance (`_inherit`) | Do not rename the standard model’s `_name` |
| Custom fields | `x_` prefix only if company policy requires it; prefer clear names without prefix when there is no collision |
| XML ID (data/views) | `own_<module>_<description>` — _e.g. `own_crm_extra_view_partner_form`_ |
| Menus / actions | Prefix aligned with the module: `own_crm_extra.menu_...` |

---

## 3. Mandatory per-module checklist

Check each item before opening a PR or installing on a shared environment.

### 3.1 Identity and packaging

- [ ] Folder at `custom/own/<module_name>/`
- [ ] Complete `__manifest__.py` (see [template](#7-manifestpy-template))
- [ ] `__init__.py` imports `models`, `controllers`, `wizard`, etc. as needed
- [ ] Module `README.md` (purpose, dependencies, configuration, screenshots if applicable)
- [ ] `static/description/index.html` (256×256 icon, Apps description)
- [ ] License aligned with company policy (`license` in manifest)

### 3.2 Dependencies (`depends`)

- [ ] Only **required** modules (do not depend on `base` “just in case” if it is already transitive)
- [ ] List explicit standard apps: _e.g. `sale`, `stock`, `website`_
- [ ] If extending website/eCommerce: include `website` / specific web modules
- [ ] Document in README if another app must be installed first

### 3.3 Security (not optional for modules with custom models)

- [ ] `security/ir.model.access.csv` — permissions per group
- [ ] `security/<module>_security.xml` — groups (`res.groups`) when applicable
- [ ] `security/ir.rule.xml` — record rules for multi-company or per-user restrictions
- [ ] Test with a user **without** administration rights
- [ ] Sensitive fields: do not expose on portal views without rules

Order in `__manifest__.py` → **security first**, then data and views:

```python
'data': [
    'security/<module>_security.xml',
    'security/ir.model.access.csv',
    # 'security/ir.rule.xml',
    'views/...',
],
```

### 3.4 Data

- [ ] `data/` — production data (sequences, crons, parameters, menus)
- [ ] `demo/` — demonstration data only; separate `'demo': [...]` from `'data'`
- [ ] XML with `noupdate="1"` only when the business requires it (avoid overwriting customer changes on upgrade)
- [ ] No credentials, internal URLs, or secrets in XML/CSV

### 3.5 Python code

- [ ] Models in `models/` with `models/__init__.py`
- [ ] `_description` on new models
- [ ] Correct `@api.constrains`, `@api.depends`, `super()` in overrides
- [ ] Avoid heavy business logic in `create`/`write` unless necessary; prefer dedicated methods
- [ ] Exceptions: `UserError` / `ValidationError` with translatable messages (`_()`)
- [ ] Odoo 18 compatibility: avoid APIs deprecated from earlier versions

### 3.6 Views and UI

- [ ] XML in `views/`; inheritance via `<xpath>` or `inherit_id` (do not duplicate full views without reason)
- [ ] `views/<module>_menus.xml` when adding menus
- [ ] Actions and menus with security groups assigned
- [ ] Chatter, smart buttons, and states aligned with standard modules you extend

### 3.7 Website / frontend (when applicable)

- [ ] `controllers/` with documented routes and appropriate `auth` (`public` / `user` / `none`)
- [ ] QWeb templates in `views/` or `static/src/xml/`
- [ ] Assets in `__manifest__.py` → `'assets'` key (bundles such as `web.assets_frontend`)
- [ ] SCSS/JS under `static/src/`; do not edit core under `odoo_18_core/`

### 3.8 Translations

- [ ] User-visible text in English in source code
- [ ] `i18n/` folder with `.pot` and `.po` per published language
- [ ] After changing strings: update translations; use `-u` with `--i18n-overwrite` only when the team agrees

### 3.9 Quality and delivery

- [ ] Tested locally in Docker (`-i` first install, `-u` after XML/data/manifest changes)
- [ ] Clean upgrade from the previous version of the same module (if it already existed in the DB)
- [ ] No `print` or debug code in commits
- [ ] CHANGELOG or README section with changes per version

---

## 4. Recommended folder structure

Adjust for module type (backend only, website, connectors, etc.).

```
custom/own/own_example/
├── __init__.py
├── __manifest__.py
├── README.md
├── controllers/          # optional — HTTP/JSON
│   ├── __init__.py
│   └── main.py
├── models/
│   ├── __init__.py
│   └── example.py
├── wizard/               # optional — transient models
│   ├── __init__.py
│   └── example_wizard.py
├── views/
│   ├── example_views.xml
│   └── example_menus.xml
├── security/
│   ├── own_example_security.xml
│   └── ir.model.access.csv
├── data/
│   └── example_data.xml
├── demo/                 # optional
├── report/               # optional — QWeb reports
├── i18n/
│   └── es.po
└── static/
    ├── description/
    │   ├── icon.png
    │   └── index.html
    └── src/
        ├── scss/
        └── js/
```

---

## 5. Business and architecture policies (fill in)

Define the rules that **all** in-house modules must follow.

| Topic | Company decision |
|-------|------------------|
| **Multi-company** | _Yes/No? Rules on `company_id`_ |
| **Multi-warehouse / multi-currency** | _Constraints or required modules_ |
| **Fields on `res.partner` / products** | _Centralize in `own_base` or per domain?_ |
| **External integrations** | _One module per API (`own_connector_x`)? Credentials via `ir.config_parameter` / environment variables_ |
| **Cron jobs** | _Naming convention, timezone, idempotency_ |
| **Email / templates** | _English only + i18n or per-language templates?_ |
| **PDF reports** | _Logo, fonts, legal footer — see brand manual if available_ |
| **Portal / eCommerce** | _What portal users may see; GDPR / consents_ |
| **Internal “base” module** | _Does `own_base` exist with shared utilities? Common `depends` list_ |

---

## 6. Module types and scope

Use this table to decide whether to create a **new module** or **extend an existing one**.

| Type | When to use | Name example |
|------|-------------|--------------|
| **Light extension** | Few fields or one view on a standard model | `own_partner_ref` |
| **Functional domain** | Full process (sales, purchases, HR) | `own_sale_approval` |
| **Website / marketing** | Pages, snippets, tracking | `own_website_brand` |
| **Connector** | REST API, EDI, banking | `own_connector_erp` |
| **Reporting** | Specific PDF/Excel | `own_account_reports` |
| **Technical / utility** | Temporary patches, migration | `own_migration_18` — **with sunset date** |

**Rule:** one module = one clear responsibility. If it grows too large, split into `own_<domain>_<feature>` with dependencies between them.

---

## 7. `__manifest__.py` template

Replace values in `<>` and remove keys you do not need.

```python
{
    'name': '<Company short name> — <Description>',
    'version': '18.0.1.0.0',
    'category': '<Sales/Website/Human Resources/...>',
    'summary': '<One line: what the module does>',
    'description': """
        <Business context>
        <Problems it solves>
        <Post-install configuration notes>
    """,
    'author': '<Author from section 1>',
    'website': '<URL>',
    'license': '<LGPL-3 or other>',
    'depends': [
        'base',
        # 'sale',
        # 'website',
    ],
    'data': [
        'security/<module>_security.xml',
        'security/ir.model.access.csv',
        'views/<module>_views.xml',
        'views/<module>_menus.xml',
    ],
    'demo': [
        # 'demo/<module>_demo.xml',
    ],
    'assets': {
        # 'web.assets_frontend': [
        #     '<module>/static/src/scss/style.scss',
        # ],
    },
    'installable': True,
    'application': False,   # True only if it is a new “app” in the main menu
    'auto_install': False,
}
```

---

## 8. Company module registry

Keep this table up to date (add a row per module in production or in development).

| Technical module | Display name | Owner | Status | DB / environments | Notes |
|------------------|--------------|-------|--------|-------------------|-------|
| `own_name_module` | _Repo example_ | _Name_ | _dev / staging / prod_ | _odoo_dev, odoo_prod_ | _Website branding_ |
| | | | | | |
| | | | | | |

---

## 9. Quick workflow: new module

1. Create folder `custom/own/<module_name>/` following [section 4](#4-recommended-folder-structure).
2. Copy and adapt the [manifest template](#7-manifestpy-template).
3. Complete the [checklist](#3-mandatory-per-module-checklist).
4. First install: `EXEC_ODOO` with `-i <module_name>` or Apps → Update Apps List → Install.
5. After view/data/manifest changes: `-u <module_name>` or Upgrade from Apps.
6. Add a row to the [module registry](#8-company-module-registry) and update the module README.

---

## 10. Useful links

| Resource | URL |
|----------|-----|
| Odoo 18 developer documentation | https://www.odoo.com/documentation/18.0/developer.html |
| Module tutorials | https://www.odoo.com/documentation/18.0/developer/tutorials.html |
| Project local guide | [`docs/local-development-guide.md`](../../docs/local-development-guide.md) |
| Odoo reference source code | `odoo_18_core/` (local clone, not versioned in Git) |

---

## One-line summary

**`own_` prefix, version `18.0.x.y.z`, security before views, English code with `i18n`, one module per responsibility, and this document plus per-module README as the source of truth for the business.**

---

*Internal document — complete [section 1](#1-company-information-fill-in-once) and [section 5](#5-business-and-architecture-policies-fill-in) with your company’s real data and review with the technical team.*
