# loutilities

The shared python library underlying scoretility, routetility, contractility, and membertility (and others). Apache-2.0 licensed, general-purpose "few hopefully useful utilities" package, but two modules are load-bearing for almost every FSRC app: `tables.py` (data grids) and `user/` (accounts, roles, multi-app SSO).

- Repo: https://github.com/louking/loutilities
- Repo's own `CLAUDE.md` (authoritative, deeper detail than this page): https://github.com/louking/loutilities/blob/master/CLAUDE.md

## tables.py — DataTables/Editor integration

The largest module (~3500 lines). Wires up server-side [DataTables](https://datatables.net/) + [Editor](https://editor.datatables.net/) grids against SQLAlchemy models — the CRUD-table pattern used throughout the admin views of every app.

Key classes:

- **`CrudApi`** — base Flask `MethodView` providing page rendering plus RESTful CRUD endpoints for a DataTables/Editor grid (serverside or client-side processing).
- **`DbCrudApi`** (extends `CrudApi`) — binds the grid directly to a SQLAlchemy model for reads/writes, with versioning and column-transform support. This is the class most app admin views subclass.
- **`DbCrudApiRolePermissions`** (extends `DbCrudApi`) — adds role-based access control to the above.
- **`DataTablesEditor`** — bidirectional mapping between DB attributes and Editor form fields (handles null ↔ empty-string, etc.).
- **`DteDbRelationship`**, **`DteDbSubrec`**, **`DteDbBool`**, **`DteDbDependent`** — field-type helpers for one-to-many/many-to-one relationships, nested subrecord fields, boolean rendering, and dependent dropdowns (options in one field driven by another field's selection).
- **`CrudFiles`** — CRUD endpoints for managing uploaded files attached to a record.
- **`TablesCsv`** — lighter-weight base view for rendering a table from CSV-like data rather than a DB-backed model.

Supporting static/template assets that consuming projects copy in: `tables-assets/`.

## user/ — accounts, roles, and multi-app SSO

Shared authentication/authorization layer built on Flask-Security-Too, used across the whole "xtility" app suite so a user's login and roles work consistently across scoretility, routetility, contractility, and membertility.

- **`model.py`** — core models: `User` (Flask-Security `UserMixin`), `Role`, `RolesUsers` (join table), `Application` (one row per app: contracts, members, routes, scores), `Interest` (see [glossary](../GLOSSARY.md)). Also `ManageLocalTables`, which syncs local copies of user/interest data across each app's own database.
- **`roles.py`** — declares the actual role catalog as data (name/description/applications) rather than code: a common `super-admin` role, plus per-app roles (e.g. eight distinct roles for the members app covering leadership, membership, meetings, racing team, awards).
- **`__init__.py`** — `UserSecurity` wraps Flask-Security to reject login/password-reset unless the user has a role for the *current* application (checked against `g.loutility`); also exposes `create_app()` to wire up the datastore.
- **`tables.py`** (`loutilities/user/tables.py`, distinct from the top-level `tables.py` above) — `DbCrudApiInterestsRolePermissions` and friends: admin CRUD grids for managing users/roles/interests themselves, filtered by interest-based access.
- **`views/userrole.py`** — the actual admin views for managing users, roles, interests, and applications; super-admins get full CRUD, ordinary admins see a restricted view limited to roles/interests they themselves are authorized to assign.
- **`tablefiles.py`** — file upload handling (`FieldUpload`, `FilesCrud`, `FilesMixin`), storing uploads in directories grouped by interest/permission level.

## Naming convention: "xtility"

The `-tility` suffix pattern (scoretility, routetility, contractility, membertility) isn't a coincidence — `loutilities/user/roles.py` refers to the app suite collectively as "xtility" in its role declarations. It's the umbrella name for apps sharing this common `loutilities`-based user/role/SSO layer.
