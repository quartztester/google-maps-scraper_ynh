# google-maps-scraper — packaging design

Go binary that drives a headless Chromium to scrape Google Maps, with a web UI and REST API. The embedded browser drives most of the packaging choices.

Anything not mentioned here is standard YunoHost boilerplate identical in most packages.

## Lifecycle

- **Install:** [download the release binary](../manifest.toml#L44-L48) (`extract=false`, renamed), then [the binary installs its own Playwright Chromium](../scripts/install#L30-L32), about 650 MB, into the data dir. Then systemd unit, nginx, start.
- **Upgrade:** re-download the binary and [re-run the browser install](../scripts/upgrade#L35-L37). That call is idempotent and fetches only what changed, since the data dir survives upgrades.
- **Backup/restore:** archive [scraped data only](../scripts/backup#L22-L24). Restore [re-downloads the browser](../scripts/restore#L35-L37) and [fixes permissions](../scripts/restore#L55-L58) that `ynh_restore` leaves broken.

## Decisions

- **Source:** [prebuilt release binary with sha256 pin](../manifest.toml#L44-L48), no build step. [amd64-only](../manifest.toml#L21) because upstream builds amd64 only.
- **Runtime:** Chromium is fetched by the app binary itself (`PLAYWRIGHT_INSTALL_ONLY=1`) into the [data dir](../scripts/_common.sh#L11), and [systemd passes it back](../conf/systemd.service#L11). Browsers in the install dir would re-download 650 MB on every upgrade, since that dir is replaced. The [long apt list](../manifest.toml#L63-L64) is Chrome's system libraries, not app deps.
- **Services:** standard, one systemd unit.
- **Storage:** standard, no database. Jobs and CSVs are plain files under `webdata`.
- **Auth:** the app has no login, so [ldap/sso are not_relevant](../manifest.toml#L24-L25) and the YunoHost permission is the only gate (default `all_users`, [ADMIN.md](ADMIN.md) warns admins to tighten it).
- **Path:** root-only. The app hardcodes absolute URLs, and the CI test for unrestricted paths is [excluded to match](../tests.toml#L3-L8).
- **Ports/firewall:** standard, one loopback port (default 8080), never exposed.
- **Scheduler:** none.
- **Migration:** none, a version bump is a binary swap.
- **Sandboxing:** [minimal on purpose](../conf/systemd.service#L19-L23), because Chromium needs user namespaces and a wide syscall surface. Isolation comes from the app user and the 127.0.0.1 bind.
- **Backup:** browser dir excluded (re-downloadable), scraped data included.
- **Admin-surface:** no config panel. Scraping options are per job in the web UI or API.

## Don't break

- [Permission fixups at the end of restore](../scripts/restore#L55-L58). Without them the restored app cannot read its own files.
- [The "minimal on purpose" comment](../conf/systemd.service#L19-L23). Hardening the unit without testing a real scrape breaks Chromium.
- nginx [50M uploads and 300s read timeout](../conf/nginx.conf#L8-L9). Jobs run for minutes; fix timeouts here, not in the app.
- The [ToS warning at install](PRE_INSTALL.md) is intentional. Scraping Google Maps may violate their terms.
