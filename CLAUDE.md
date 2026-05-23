# Project guide for Claude — `django-superapp-admin-portal`

Copier template for the **`admin_portal` superapp** — the centralized admin entrypoint built on `django-unfold`. Every downstream project that uses `django-superapp` consumes this app via:

```bash
django_superapp bootstrap-app --template-repo https://github.com/superapp-labs/django-superapp-admin-portal ./admin_portal
```

**Read [`./README.md`](./README.md) and the upstream [`../django-superapp/CONVENTIONS.md`](https://github.com/superapp-labs/django-superapp/blob/main/CONVENTIONS.md) first.** This file is the contributor brief.

## Hard rules when contributing

1. **Native `django-unfold` only — no exceptions.** This app ships `SuperAppModelAdmin` (extends `unfold.admin.ModelAdmin`) and `superapp_admin_site` (extends `unfold.sites.UnfoldAdminSite`). Downstream code is required to inherit from them. If something doesn't fit Unfold, *extend Unfold's class*, don't write a parallel admin path. Anti-patterns to refuse on sight:
   - `from django.contrib import admin; admin.site.register(...)` — must be `@admin.register(Foo, site=superapp_admin_site)`
   - Hand-rolled changelist templates (Unfold's filters / inlines / actions cover every standard case)
   - Custom CSS to "fix" Unfold layout — bump Unfold first, file an upstream bug second
2. **Imports must track Unfold's public API.** Unfold reorganises modules between minor versions (`UnfoldAdminReadonlyField` moved from `unfold.admin` to `unfold.fields` in 0.94). When a downstream pin update breaks `helpers.py` / `admin.py`, fix the import here — not in every downstream project.
3. **One class per file.** Already enforced by `CONVENTIONS.md` in the parent toolkit. `admin.py` lives as a single file in this template by tradition (it's the base classes), but downstream `<app>/admin/<slug>.py` MUST be one ModelAdmin per file. Document the convention in `README.md` if a contributor seems to be drifting.
4. **Keep `requirements.txt` pinned to a known-working Unfold minor.** When bumping, run the downstream default-project + a real consumer (e.g. the `awesome-repositories.com` backend) to catch import / template regressions before merging. The current pin is `django-unfold==0.94.*` — track it forward as Unfold releases new minors.
5. **Sidebar nav lives downstream, not here.** This template only initializes `UNFOLD['SIDEBAR'] = { 'show_search': False, 'show_all_applications': True, 'navigation': [] }` (empty list). Downstream apps append their own sections via their `<app>/settings.py extend_superapp_settings(main_settings)`. Do not add app-specific nav items to this template.

## Common edits

- **Bump Unfold** → update `requirements.txt`. Then sweep `helpers.py`, `admin.py`, `sites.py`, `widgets.py`, `forms.py`, `decorators.py` for moved imports. Run a real consumer's `manage.py check`.
- **Add a shared admin mixin / decorator** → put it in `admin.py` (mixin) or `decorators.py` (function). Re-export from `helpers.py` if downstream `SuperAppModelAdmin = ...` users rely on it.
- **Change site behavior** → `sites.py` (`SuperAppAdminSite(UnfoldAdminSite)`). Keep overrides minimal — every override is something Unfold could break next release.
- **Tailwind / CSS** → `tailwind/` directory. Build via the Makefile target in the default-project template.

## What downstream consumers depend on

A change here ripples through every project bootstrapped from `django-superapp-default-project`. Treat these as the **public API**:

- `superapp.apps.admin_portal.admin.SuperAppModelAdmin`
- `superapp.apps.admin_portal.sites.superapp_admin_site`
- `superapp.apps.admin_portal.helpers.SuperAppAdminReadonlyField`
- The shape of `UNFOLD` in `settings.py` (downstream apps mutate `UNFOLD['SIDEBAR']['navigation']`)

Renames are breaking. If you must rename, leave a one-release shim and document the migration.

## Companion files for agents

`AGENTS.md` is a symlink to this file so agent tools that look for either filename pick up the same rules.
