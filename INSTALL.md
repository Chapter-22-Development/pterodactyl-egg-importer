# Installation / Upgrade — Nest-Scoped Egg Catalog Importer

These steps target a generic **Pterodactyl Panel 1.x** install. Paths below are relative to your panel root (commonly `/var/www/pterodactyl`).

> Prefer **merging** Blade and route changes rather than blindly overwriting customized theme files. Full copies are provided under `src/` for reference; unified diffs are under `patches/`.

## 1. Copy PHP files

```bash
# From the extracted package directory:
mkdir -p /var/www/pterodactyl/app/Services/MassEggImporter

cp src/app/Services/MassEggImporter/MassEggImporterService.php \
  /var/www/pterodactyl/app/Services/MassEggImporter/

cp src/app/Http/Controllers/Admin/Nests/NestEggCatalogController.php \
  /var/www/pterodactyl/app/Http/Controllers/Admin/Nests/
```

## 2. Add routes

**Option A — patch** (against a stock-like `routes/admin.php`):

```bash
cd /var/www/pterodactyl
patch -p1 < /path/to/package/patches/routes-admin-egg-catalog.patch
```

**Option B — manual merge:** open `routes/admin.php`, find the nests route group, and after the existing `admin.nests.egg.import` route insert the four routes from `src/routes/egg-catalog-routes.php` (the `Route::get/post ... egg-catalog ...` lines only — do not paste the `use` lines if they already exist).

Example insertion:

```php
Route::get('/view/{nest:id}/egg-catalog', [Admin\Nests\NestEggCatalogController::class, 'catalogue'])->name('admin.nests.egg-catalog.catalogue');
Route::post('/view/{nest:id}/egg-catalog/import', [Admin\Nests\NestEggCatalogController::class, 'import'])->name('admin.nests.egg-catalog.import');
Route::post('/view/{nest:id}/egg-catalog/import-bulk', [Admin\Nests\NestEggCatalogController::class, 'importBulk'])->name('admin.nests.egg-catalog.import-bulk');
Route::post('/view/{nest:id}/egg-catalog/import-files', [Admin\Nests\NestEggCatalogController::class, 'importFiles'])->name('admin.nests.egg-catalog.import-files');
```

## 3. Update nest views

### Nest view (required — catalog modal)

If your `resources/views/admin/nests/view.blade.php` is still stock:

```bash
cd /var/www/pterodactyl
patch -p1 < /path/to/package/patches/view.blade.php.patch
# OR replace with the packaged full file (only if you have no local customizations):
# cp /path/to/package/src/resources/views/admin/nests/view.blade.php resources/views/admin/nests/view.blade.php
```

If your nest view is customized, merge manually:

1. Add the **Import from Catalog** button in the "Nest Eggs" box header (`#eggCatalogModal` trigger).
2. Paste the modal markup and footer JavaScript from the packaged `view.blade.php` (everything from `{{-- Nest-scoped egg catalog importer --}}` through the end of the catalog `<script>` IIFE). Keep your existing delete-button hover script.

### Nests index tip (optional)

```bash
cd /var/www/pterodactyl
patch -p1 < /path/to/package/patches/index.blade.php.patch
```

Or add a small tip next to the Import Egg button noting that nest view → **Import from Catalog** supports eggs.pterodactyl.io / GitHub JSON.

## 4. Optional config

```bash
cp src/config/egg_catalog.php /var/www/pterodactyl/config/egg_catalog.php
```

Env overrides (optional):

```env
EGG_CATALOG_CATEGORIES_URL=https://eggs.pterodactyl.io/api/categories.json
EGG_CATALOG_EGGS_URL=https://eggs.pterodactyl.io/api/eggs.json
EGG_CATALOG_TIMEOUT=20
```

Defaults already point at the public catalogue; you normally do not need to set these. Do **not** hardcode private panel hostnames here.

## 5. Permissions, cache, PHP-FPM

```bash
cd /var/www/pterodactyl
chown -R www-data:www-data app/Services/MassEggImporter \
  app/Http/Controllers/Admin/Nests/NestEggCatalogController.php \
  resources/views/admin/nests/view.blade.php \
  resources/views/admin/nests/index.blade.php
# Adjust user/group if your panel does not use www-data.

php artisan route:clear
php artisan view:clear
php artisan config:clear
# If you use config/route caching in production:
# php artisan config:cache
# php artisan route:cache

# Optional — reload PHP-FPM so opcache picks up new classes:
systemctl reload php8.2-fpm || systemctl reload php8.1-fpm || true
```

## 6. Verify

1. Log into the admin area.
2. Open **Nests → (any nest)**.
3. Confirm **Import from Catalog** appears and the modal loads the catalogue.
4. Import a single test egg (without replace) and confirm it appears in that nest only.

## Upgrade

Re-copy the PHP service + controller, re-apply Blade/route merges if your copies diverged, then clear caches / reload FPM as in step 5.

## Uninstall

1. Remove the four `egg-catalog` routes from `routes/admin.php`.
2. Remove catalog modal / button / JS from `resources/views/admin/nests/view.blade.php` (and the index tip if added).
3. Delete:
   - `app/Http/Controllers/Admin/Nests/NestEggCatalogController.php`
   - `app/Services/MassEggImporter/MassEggImporterService.php` (and the directory if empty)
   - `config/egg_catalog.php` if you added it
4. Clear caches again.

## Notes

- This package does **not** require Theme / Luna mass-egg settings UI. Nest-scoped routes + modal are sufficient.
- Example private panels (e.g. a host's own panel URL) are irrelevant to configuration; catalogue traffic goes to eggs.pterodactyl.io / GitHub only.
