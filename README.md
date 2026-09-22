# Pterodactyl Nest-Scoped Egg Catalog Importer
Import eggs from the public eggs.pterodactyl.io catalogue (and allowlisted GitHub raw/blob JSON URLs) directly into a specific nest from the Pterodactyl admin UI.

Built by Chapter22 Development. Released under the MIT License.

# What it does
On Admin → Nests → (open a nest) you get an Import from Catalog button that opens a modal to:

Browse the public catalogue (categories + search)
Import one or many eggs into that nest only
Paste an allowlisted egg JSON URL (GitHub raw/blob or eggs.pterodactyl.io)
Optionally replace an existing egg with the same name
(API also supports uploading egg JSON files into the nest)
Fetches happen server-side. Only HTTPS URLs on a fixed host allowlist are accepted:

eggs.pterodactyl.io
raw.githubusercontent.com
github.com (blob/raw repository file paths only)
objects.githubusercontent.com
Requirements
Pterodactyl Panel 1.x (PHP 8.1+/8.2+ as required by your panel version)
Outbound HTTPS from the panel host to the catalogue / GitHub
Standard panel egg import services (EggImporterService, EggUpdateImporterService) — already present in stock Pterodactyl
Optional:

config/egg_catalog.php (included) to override catalogue URLs / timeout via env
If a Luna-style ThemeSettings model exists, addon settings under addons.mass_egg_importer are also honored; stock panels do not need this
Usage
Install using INSTALL.md.
Open Admin → Nests, click a nest.
Click Import from Catalog.
Browse / search, select eggs, import (or paste a JSON URL on the second tab).
The nests index page also shows a short tip pointing admins at the nest view catalog button.

# Package layout
src/app/Services/MassEggImporter/MassEggImporterService.php
src/app/Http/Controllers/Admin/Nests/NestEggCatalogController.php
src/resources/views/admin/nests/view.blade.php      # full nest view with catalog modal
src/resources/views/admin/nests/index.blade.php     # nests list + tip (merge carefully)
src/routes/egg-catalog-routes.php                  # route snippet to merge into routes/admin.php
src/config/egg_catalog.php                         # optional config
patches/                                           # unified diffs vs stock nest views / routes

Theme/settings mass-importer hooks (if present on some themed panels) are out of scope. This package is the nest-scoped path only.

# Security notes
Download URLs are validated against an allowlist (HTTPS + host + path rules for github.com).
Imports reuse Pterodactyl’s existing egg importer / update-importer services.
Admin authentication / CSRF apply as with other admin routes.
Demo panels that set IS_DEMO=true refuse these endpoints (403).
# License
MIT — see LICENSE. Copyright (c) 2026 Chapter22 Development.
