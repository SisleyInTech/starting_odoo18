# Odoo 18 — core source code

This directory (`odoo_18_core/`) is reserved for the **Odoo 18 source code**. **Download or clone** the official Odoo repository on the branch that matches version 18 so the project tree lives **inside this folder** (for example, after `git clone` you typically get `odoo_18_core/odoo/` or another subdirectory you choose, always under `odoo_18_core/`).

## What to do

1. Go into `odoo_18_core/`.
2. Clone or extract Odoo 18 following the [official Odoo documentation](https://www.odoo.com/documentation/18.0/) or your team’s workflow (Git, archive, etc.).
3. Do not commit that code to the `starting_odoo18` repository: `.gitignore` is set so downloaded content under this folder is **not versioned**; only this instructions file (`odoo_18_core.md`) stays in Git.

## Note

If you change the layout (clone subdirectory name, multiple checkouts, etc.), the important part is that **all Odoo 18 code** stays under `odoo_18_core/` and you do not mix that tree with the project’s own modules outside this folder unless your architecture says otherwise.
