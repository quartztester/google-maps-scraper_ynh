# Design decisions

Why this package is structured the way it is. Each section links the code.

## Prebuilt binary

- Upstream ships a prebuilt Linux binary; the package downloads it and pins the sha256. No build step exists here.
- Upstream builds amd64 only, so the package is amd64 only (no Raspberry Pi).

Code: [manifest.toml L44-L47](../manifest.toml#L44-L47), [scripts/install L16](../scripts/install#L16), [manifest.toml L21](../manifest.toml#L21)

## Chromium browser

- The scraper drives a headless Chrome browser; install downloads ~650 MB of it.
- Browser files go in the data dir, not the install dir, because upgrades replace the install dir. This avoids re-downloading 650 MB per upgrade.
- Backups exclude the browser dir (re-downloadable); restore re-runs the browser install. Only scraped data (jobs, CSVs) is backed up.
- The long apt list in the manifest is Chrome's system libraries, not app dependencies. Missing-library errors at job time get fixed in that list.
- Restore ends with a permission fixup because YunoHost restore leaves files unreadable to the app. Do not remove it.

Code: [scripts/install L30-L32](../scripts/install#L30-L32), [scripts/_common.sh L11](../scripts/_common.sh#L11), [conf/systemd.service L11](../conf/systemd.service#L11), [scripts/backup L22-L24](../scripts/backup#L22-L24), [scripts/restore L35-L37](../scripts/restore#L35-L37), [scripts/restore L55-L58](../scripts/restore#L55-L58), [manifest.toml L63-L64](../manifest.toml#L63-L64)

## Domain root only (`path = /`)

- The app hardcodes absolute URLs from the domain root. A sub-path install completes but the UI is broken.
- Each extra instance therefore needs its own domain; CI test for unrestricted paths is excluded on purpose.

Code: [tests.toml L4-L8](../tests.toml#L4-L8), [doc/ADMIN.md](ADMIN.md)

## Authentication

- The app has no login of its own. Access is controlled only by the YunoHost permission on the URL, default `all_users`. Anyone with access can run scrapes and download results.

Code: [manifest.toml L24-L25](../manifest.toml#L24-L25)

## Weak systemd sandboxing (on purpose)

- Strict sandboxing breaks Chrome (needs user namespaces and a wide syscall surface), causing crashes that look like app bugs.
- Isolation comes instead from the dedicated app user and binding to 127.0.0.1; nginx + SSO is the only exposed surface.
- Do not harden the unit file without testing a real scrape after.

Code: [conf/systemd.service L18-L23](../conf/systemd.service#L18-L23)

## Misc

- nginx: 50 MB upload limit, 300 s read timeout — scrapes run for minutes and CSVs are large. Job timeouts get fixed here, not in the app.
- The ToS warning in PRE_INSTALL.md is intentional; scraping Google Maps may violate Google's terms.
- RAM/disk figures in the manifest are sized for Chrome: 1 GB runtime, 1 GB disk.

Code: [conf/nginx.conf L8-L9](../conf/nginx.conf#L8-L9), [doc/PRE_INSTALL.md](PRE_INSTALL.md), [manifest.toml L27-L29](../manifest.toml#L27-L29)
