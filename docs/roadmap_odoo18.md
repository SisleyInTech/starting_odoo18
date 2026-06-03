# Odoo 18 Development Roadmap

This document is a practical roadmap for learning, building, and maintaining Odoo 18 modules in this repository. It complements the setup instructions in [`docs/local-development-guide.md`](local-development-guide.md) and the in-house module standards in [`custom/own/own-module-guide.md`](../custom/own/own-module-guide.md).

---

## 1. Purpose

Use this roadmap to guide developers from basic Odoo concepts to production-ready custom modules.

The goal is to:

- Understand how Odoo 18 works internally.
- Build custom modules without modifying the Odoo core.
- Follow this repository's conventions for `custom/own/`.
- Deliver modules that are secure, maintainable, translatable, and upgrade-friendly.

---

## 2. Project Facts To Keep In Sync

These values reflect the current repository configuration.

| Area | Current value |
|------|---------------|
| Odoo image | `odoo:18.0` |
| Development container | `own-odoo18-dev` |
| Local URL | `http://localhost:8049/` |
| Compose file | `docker/docker-compose-dev.yml` |
| Development config | `config/odoo-dev.conf` |
| Addons path inside container | `/mnt/extra-addons/custom/own` |
| In-house modules folder | `custom/own/` |
| Module prefix | `own_` |
| Docker network | `own_network` |
| Optional Odoo source reference | `odoo_18_core/` |

Important: `odoo_18_core/` is only for reading and searching the official Odoo source locally. Do not customize business logic there.

---

## 3. Core Principles

### 3.1 Do Not Modify Odoo Core

Odoo is designed to be extended through modules. Modifying the core makes upgrades harder, creates hidden conflicts, and complicates support.

Instead:

- Extend existing models with `_inherit`.
- Add new models in custom modules when the business concept is new.
- Inherit and modify views using XML inheritance and `xpath`.
- Add data, security, reports, controllers, and assets from your own module.

### 3.2 One Module, One Responsibility

Each module should solve one clear business or technical problem.

Good examples:

- `own_partner_fiscal_id`
- `own_sale_approval`
- `own_website_brand`
- `own_connector_external_api`

Avoid broad or vague modules such as `own_utils`, `own_fixes`, or `own_customizations`.

### 3.3 Security Comes Before UI

Every custom model must define access rights before exposing menus, views, or controllers.

Minimum security files:

- `security/ir.model.access.csv`
- `security/<module>_security.xml` when custom groups are needed
- `security/ir.rule.xml` when records must be restricted by company, user, team, or ownership

### 3.4 English Source, Translated Output

Code, XML strings, labels, and README content should be written in English. User-facing translations belong in `i18n/*.po`.

---

## 4. How Odoo Works

### 4.1 Modular Architecture

Odoo functionality is organized into modules. A module can add or extend:

- Models and fields
- Views and menus
- Actions and server actions
- Security groups and access rights
- Reports
- Website controllers and templates
- Scheduled actions
- Demo or initial data
- Frontend/backend assets

Modules are installed, upgraded, and uninstalled independently, but dependencies define the order in which they are loaded.

### 4.2 Module Loading

Odoo reads each module through its `__manifest__.py`. The manifest tells Odoo:

- The module name, version, author, license, and category.
- Which modules must be installed first (`depends`).
- Which XML/CSV files must be loaded (`data`, `demo`).
- Which frontend/backend assets must be included (`assets`).
- Whether the module is installable, auto-installed, or displayed as an application.

When a module is installed or upgraded, Odoo loads Python code, applies model changes, loads security, imports data, updates views, and refreshes metadata.

### 4.3 Development Cycle

Typical local loop:

1. Edit files under `custom/own/<module_name>/`.
2. Restart the container if needed for Python reload issues.
3. Upgrade the module after XML, data, security, asset, or manifest changes.
4. Test with an administrator and with a normal business user.
5. Update README, changelog, and translations when needed.

Example `EXEC_ODOO` for module upgrade:

```env
EXEC_ODOO=--db_host=${POSTGRES_HOST} --db_port=${POSTGRES_PORT} --db_user=${POSTGRES_USER} --db_password=${POSTGRES_PASSWORD} -d ${POSTGRES_DB} -u own_my_module -c /var/lib/odoo/odoo.runtime.conf
```

---

## 5. Roadmap Phases

### Phase 0: Environment Readiness

Outcome: the developer can run Odoo locally and understand where files live.

Checklist:

- [ ] Docker and Docker Compose are installed.
- [ ] PostgreSQL is reachable from `own_network`.
- [ ] `docker/.env` exists and is not committed.
- [ ] `ADMIN_PASSWD` is configured.
- [ ] Odoo starts at `http://localhost:8049/`.
- [ ] The developer can follow logs from `own-odoo18-dev`.
- [ ] Optional: Odoo 18 source is cloned under `odoo_18_core/`.

Useful commands:

```bash
cd docker
docker compose -f docker-compose-dev.yml up -d
docker compose -f docker-compose-dev.yml logs -f own-odoo18-dev
docker exec -it own-odoo18-dev bash
```

Reference: [`README.md`](../README.md).

### Phase 1: Odoo Fundamentals

Outcome: the developer understands the basic building blocks of Odoo.

Learn:

- Models, fields, and recordsets.
- ORM methods: `create`, `write`, `unlink`, `search`, `browse`, `read_group`.
- Computed fields, constraints, onchange methods, and defaults.
- XML data files and external IDs.
- Menus, actions, and views.
- Access rights and record rules.
- Module installation and upgrade.

Deliverable:

- A small training module under `custom/own/own_training_basics/` with one model, one menu, one action, form/list views, and access rights.

### Phase 2: Backend UI

Outcome: the developer can build clean backend views that feel native to Odoo.

Learn:

- Form views with `sheet`, `group`, `notebook`, `header`, and statusbar.
- List views for fast record browsing.
- Search views with filters, group-by options, and default contexts.
- Kanban views for visual workflows.
- Smart buttons and chatter (`mail.thread`) when business users need traceability.
- View inheritance using `inherit_id` and `xpath`.

Example action and menus:

```xml
<record id="action_own_training_member" model="ir.actions.act_window">
    <field name="name">Members</field>
    <field name="res_model">own.training.member</field>
    <field name="view_mode">list,form</field>
</record>

<menuitem id="menu_own_training_root" name="Training" sequence="10"/>
<menuitem id="menu_own_training_member"
          name="Members"
          parent="menu_own_training_root"
          action="action_own_training_member"/>
```

Example form:

```xml
<form string="Member">
    <sheet>
        <group>
            <field name="name"/>
            <field name="birthdate"/>
            <field name="signup_date"/>
            <field name="active"/>
        </group>
    </sheet>
</form>
```

Deliverable:

- A backend module with list, form, search, and kanban views, including a menu hierarchy and user-friendly labels.

### Phase 3: Business Logic

Outcome: the developer can implement business rules safely.

Learn:

- Model inheritance with `_inherit`.
- New business models with `_name`.
- Computed and stored fields.
- SQL constraints and Python constraints.
- Server actions and scheduled actions (`ir.cron`).
- Wizards with `TransientModel`.
- Business exceptions with `UserError` and `ValidationError`.

Example constraint:

```python
from odoo import api, fields, models, _
from odoo.exceptions import ValidationError


class OwnTrainingMember(models.Model):
    _name = 'own.training.member'
    _description = 'Training Member'

    name = fields.Char(required=True)
    birthdate = fields.Date()

    @api.constrains('birthdate')
    def _check_birthdate(self):
        for record in self:
            if record.birthdate and record.birthdate > fields.Date.today():
                raise ValidationError(_('Birthdate cannot be in the future.'))
```

Deliverable:

- A module that enforces at least one real business rule and handles invalid data with translatable errors.

### Phase 4: Security And Multi-Company

Outcome: the developer can protect data correctly.

Learn:

- Access rights in `ir.model.access.csv`.
- Functional groups with `res.groups`.
- Record rules with `ir.rule`.
- Multi-company fields and domains.
- Portal/public access boundaries.
- Testing with non-admin users.

Minimum access example:

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_own_training_member_user,own.training.member.user,model_own_training_member,base.group_user,1,1,1,0
```

Deliverable:

- A module where at least two user profiles have different permissions, tested without administrator rights.

### Phase 5: Website, Portal, And Assets

Outcome: the developer understands when and how to expose data outside the backend.

Learn:

- HTTP controllers and route authentication (`public`, `user`, `none`).
- QWeb templates.
- Website pages and snippets.
- Portal access.
- Assets in `web.assets_frontend` or other bundles.
- Safe handling of public data and forms.

Deliverable:

- A small website or portal feature with explicit access rules and no sensitive data leakage.

### Phase 6: Reports, Email, And Automation

Outcome: the developer can automate operational workflows.

Learn:

- QWeb PDF reports.
- Mail templates.
- Activities and chatter.
- Scheduled actions.
- Import/export helpers.
- Configuration through `res.config.settings` and `ir.config_parameter`.

Deliverable:

- A feature that creates a report, sends or prepares an email, and has a documented configuration path.

### Phase 7: Quality, Delivery, And Maintenance

Outcome: the developer can ship modules that survive upgrades.

Learn:

- Module versioning: `18.0.x.y.z`.
- Clean upgrades using `-u <module_name>`.
- Data migration considerations.
- Changelog and README maintenance.
- Translation updates.
- Manual test scripts for critical flows.

Release checklist:

- [ ] `__manifest__.py` version updated when behavior changes.
- [ ] Security files are loaded before views.
- [ ] Module upgrades cleanly on an existing database.
- [ ] Normal users can complete the expected flows.
- [ ] README and module registry are updated.
- [ ] User-facing strings are translated if required.

---

## 6. Recommended Module Structure

Use this structure as the default starting point:

```text
custom/own/own_example/
+-- __init__.py
+-- __manifest__.py
+-- README.md
+-- models/
|   +-- __init__.py
|   +-- example.py
+-- views/
|   +-- example_views.xml
|   +-- example_menus.xml
+-- security/
|   +-- own_example_security.xml
|   +-- ir.model.access.csv
+-- data/
|   +-- example_data.xml
+-- i18n/
|   +-- es.po
+-- static/
    +-- description/
        +-- icon.png
        +-- index.html
```

For the full checklist, use [`custom/own/own-module-guide.md`](../custom/own/own-module-guide.md).

---

## 7. Suggested Learning Order

| Step | Topic | Why it matters |
|------|-------|----------------|
| 1 | Environment and module install/upgrade | Everything depends on a stable local loop |
| 2 | Models and fields | Data structure is the base of every Odoo module |
| 3 | Views, actions, and menus | Users interact with data through the UI |
| 4 | Security | Data must be protected before it is exposed |
| 5 | Business logic | Rules must live in maintainable Python methods |
| 6 | Reports and automation | Most business value comes from repeatable workflows |
| 7 | Website/portal | Public surfaces require stricter access thinking |
| 8 | Translations and maintenance | Production modules must be understandable and upgradeable |

---

## 8. Common Mistakes To Avoid

- Editing files inside `odoo_18_core/` for business customizations.
- Adding views before access rights.
- Using admin-only testing as proof that a module works.
- Creating broad modules with unrelated features.
- Hardcoding credentials, URLs, company IDs, user IDs, or database names.
- Copying full standard views instead of using XML inheritance.
- Leaving Spanish or another target language directly in source strings.
- Forgetting to upgrade the module after XML, security, manifest, or data changes.
- Using `noupdate="1"` without a clear reason.
- Exposing backend-only data through public controllers or portal templates.

---

## 9. Roadmap Completion Criteria

A developer is ready to work on production modules when they can:

- Create a module under `custom/own/` using the `own_` prefix.
- Define a correct `__manifest__.py`.
- Add models, views, actions, menus, and security files.
- Upgrade the module using `EXEC_ODOO` or the Apps UI.
- Explain when to use `_inherit` versus `_name`.
- Test with both admin and non-admin users.
- Keep user-facing strings translatable.
- Document the module purpose, configuration, and release notes.

---

## 10. References

| Resource | Location |
|----------|----------|
| Project README | [`README.md`](../README.md) |
| Local development guide | [`docs/local-development-guide.md`](local-development-guide.md) |
| In-house module guide | [`custom/own/own-module-guide.md`](../custom/own/own-module-guide.md) |
| Odoo core source instructions | [`odoo_18_core/odoo_18_core.md`](../odoo_18_core/odoo_18_core.md) |
| Official Odoo 18 documentation | https://www.odoo.com/documentation/18.0/ |

---

## One-Line Summary

Learn Odoo 18 by building small, secure, upgradeable modules in `custom/own/`: start with models and views, add security early, keep source text in English, never modify core, and document every production module.