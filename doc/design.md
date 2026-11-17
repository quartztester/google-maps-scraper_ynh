# Design decisions

Why this package is structured the way it is. Links go to the exact code.

## Prebuilt binary

- Upstream ships a [prebuilt Linux binary](../manifest.toml#L44-L47); the package downloads it and pins the sha256. No build step exists here — just a [one-line deploy](../scripts/install#L16).
- Upstream builds amd64 only, so the package is [amd64 only](../manifest.toml#L21) (no Raspberry Pi).

## Chromium browser

- The scraper drives a headless Chrome browser; install [downloads ~650 MB of it](../scripts/install#L30-L32).
- Browser files go in the [data dir](../scripts/_common.sh#L11), not the install dir, because upgrades replace the install dir. This avoids re-downloading 650 MB per upgrade. The [systemd unit](../conf/systemd.service#L11) points the app at that dir.
- [Backups exclude the browser dir](../scripts/backup#L22-L24) (re-downloadable); [restore re-runs the browser install](../scripts/restore#L35-L37). Only scraped data (jobs, CSVs) is backed up.
- The [long apt list](../manifest.toml#L63-L64) is Chrome's system libraries, not app dependencies. Missing-library errors at job time get fixed in that list.
- Restore ends with a [permission fixup](../scripts/restore#L55-L58) because YunoHost restore leaves files unreadable to the app. Do not remove it.

## Domain root only (`path = /`)

- The app hardcodes absolute URLs from the domain root. A sub-path install completes but the UI is broken ([ADMIN.md](ADMIN.md)).
- Each extra instance therefore needs its own domain; the CI test for unrestricted paths is [excluded on purpose](../tests.toml#L4-L8).

## Authentication

- The app has no login of its own ([ldap/sso not relevant](../manifest.toml#L24-L25)). Access is controlled only by the YunoHost permission on the URL, default `all_users`. Anyone with access can run scrapes and download results.

## Weak systemd sandboxing (on purpose)

- [Strict sandboxing breaks Chrome](../conf/systemd.service#L18-L23) (needs user namespaces and a wide syscall surface), causing crashes that look like app bugs.
- Isolation comes instead from the dedicated app user and binding to 127.0.0.1; nginx + SSO is the only exposed surface.
- Do not harden the unit file without testing a real scrape after.

## Misc

- nginx: [50 MB upload limit, 300 s read timeout](../conf/nginx.conf#L8-L9) — scrapes run for minutes and CSVs are large. Job timeouts get fixed here, not in the app.
- The [ToS warning](PRE_INSTALL.md) at install time is intentional; scraping Google Maps may violate Google's terms.
- [RAM/disk figures](../manifest.toml#L27-L29) are sized for Chrome: 1 GB runtime, 1 GB disk.
