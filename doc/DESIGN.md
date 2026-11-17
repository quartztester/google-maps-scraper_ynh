# google-maps-scraper — packaging design

Go binary that drives a headless Chromium to scrape Google Maps, web UI + REST API. The embedded browser shapes everything below.

Anything not mentioned here is standard YunoHost boilerplate identical in most packages.

## Lifecycle

- **Install:** download the single [release binary](../manifest.toml#L44-L47) (`extract=false`, renamed), then [the binary installs its own Playwright Chromium](../scripts/install#L30-L32) (~650 MB) into the data dir → systemd + nginx → start.
- **Upgrade:** re-download binary, [re-run the browser install](../scripts/upgrade#L52-L57) — idempotent, fetches only what changed, since the data dir survives upgrades.
- **Backup/restore:** archive [scraped data only](../scripts/backup#L22-L24); restore [re-downloads the browser](../scripts/restore#L35-L37) and [fixes permissions](../scripts/restore#L55-L58) `ynh_restore` leaves broken.

## Decisions

- **Source:** [prebuilt release binary + sha256](../manifest.toml#L44-L47), no build step. amd64-only because upstream builds amd64 only ([manifest L21](../manifest.toml#L21)).
- **Runtime:** Chromium fetched by the app binary itself (`PLAYWRIGHT_INSTALL_ONLY=1`) into the [data dir](../scripts/_common.sh#L11), passed back via [systemd env](../conf/systemd.service#L11) — the install dir is replaced on every upgrade, so browsers there would re-download 650 MB per version. The [long apt list](../manifest.toml#L63-L64) is Chrome's system libraries, not app deps.
- **Storage:** standard — no DB, jobs and CSVs are plain files in the data dir.
- **Auth:** standard — the app has no login; [ldap/sso not_relevant](../manifest.toml#L24-L25), access is the YunoHost permission alone (default `all_users`, see [ADMIN.md](ADMIN.md)).
- **Path:** root-only — the app hardcodes absolute URLs; the CI subdir test is [excluded to match](../tests.toml#L4-L9).
- **Sandboxing:** [minimal on purpose](../conf/systemd.service#L19-L23) — Chromium needs user namespaces and a wide syscall surface; isolation comes from the app user + 127.0.0.1 bind.
- **Backup:** browser excluded (re-downloadable); scraped data included.

## Don't break

- [Permission fixups at the end of restore](../scripts/restore#L55-L58) — without them the restored app can't read its own files.
- [The "minimal on purpose" comment](../conf/systemd.service#L19-L23) — hardening the unit without testing a real scrape breaks Chrome.
- nginx [50M uploads / 300s read timeout](../conf/nginx.conf#L8-L9) — jobs run for minutes; fix timeouts here, not in the app.
- The [ToS warning at install](PRE_INSTALL.md) is intentional: scraping Google Maps may violate their terms.
