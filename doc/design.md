# Design decisions — Google Maps Scraper YunoHost package

For future maintainers: this explains *why* the package is built the way it is.
Each section names the files to look at if you want to see the code.
Upstream app: https://github.com/gosom/google-maps-scraper (Go binary that
drives a headless Chromium to scrape Google Maps, with a web UI + REST API).

## 1. Ship the prebuilt binary, don't build from source

**Decision:** Download upstream's release binary as-is. In `manifest.toml`,
`[resources.sources.main]` has `extract = false` and `rename = "gmaps"`, so
`ynh_setup_source` drops the single file straight into `/var/www/$app/gmaps`.

**Why:** Upstream already ships a static Go binary per release. There is no
compile step to reproduce, no Go toolchain or node build to maintain, and the
SHA256 pin gives supply-chain integrity. Building from source would add a
build stage that produces a byte-identical artifact for months of extra
packaging code.

**How to upgrade:** change `version` in `manifest.toml`, update the release
`url` + `sha256` in `[resources.sources.main]` (grab both from the upstream
release page), bump the `~ynhN` suffix if you changed packaging but not the
upstream version. No install-script changes needed — `install` and `upgrade`
share the same `ynh_setup_source` step.

**Consequence:** `architectures = ["amd64"]` only, because upstream only
builds amd64 binaries. ARM users can't run this package; that's upstream's
limit, not ours.

## 2. Chromium is installed at install time, stored in the data dir

**Decision:** The scraper needs a headless Chromium (Playwright). Instead of
bundling it in the download or installing a system `chromium` package, we ask
the app binary itself to fetch it:

```bash
ynh_exec_as_app env PLAYWRIGHT_INSTALL_ONLY=1 \
    PLAYWRIGHT_BROWSERS_PATH="$data_dir/browsers" \
    "$install_dir/gmaps"
```

`PLAYWRIGHT_INSTALL_ONLY=1` makes the binary install its browser and exit —
it doubles as the installer. The ~650 MB of browser files land in
`$data_dir/browsers` (`browsers_dir` in `scripts/_common.sh`), and the same
env var is set in `conf/systemd.service` so the running app finds them.

**Why the data dir and not `/var/www/$app`:** the install dir is replaced on
every upgrade; the data dir isn't. Keeping the browser in the data dir means
upgrading the app version doesn't re-download 650 MB every time — the
re-install is idempotent and only fetches what changed. We avoid the system
Chromium because Playwright pins browser builds to the driver version baked
into the binary; mixing versions is a classic source of silent scrape
failures.

**Why the long apt list in `manifest.toml`:** those are *not* app
dependencies — they are the shared libraries Chromium links against
(`libnss3`, `libgbm1`, `libasound2`, ...). Debian won't install them on its
own; without one of them the browser fails at job time with a cryptic
error, not at install time. If a Chromium-related job failure ever appears,
check this list first.

## 3. Back up the scraped data, not the browser

**Decision:** `scripts/backup` backs up `$data_dir/webdata` (the jobs DB and
CSV results) but deliberately skips `$data_dir/browsers`. `scripts/restore`
re-runs the Playwright install step to re-fetch the browser.

**Why:** 650 MB of re-downloadable vendor binaries would bloat every backup
archive for zero restore value — the restore script can always rebuild them
(it's the same command install uses). Backup size is a real cost on small
YunoHost boxes.

**Gotcha in restore:** `ynh_restore` chmods restored non-root paths to 0400,
which breaks the running app; the end of `scripts/restore` fixes ownership and
modes in one pass. Don't remove that block when refactoring.

## 4. Forced to the domain root (`path = /`)

**Decision:** `tests.toml` pins `path = "/"` and excludes the
`multi_instance_and_domain_not_constrained` CI test; `doc/ADMIN.md` states it
as an admin constraint.

**Why:** the web UI and API hardcode absolute URLs from the domain root, so
installing under `/gmaps` gives a broken UI. This isn't a packaging
preference — a sub-path install *looks* like it works (nginx proxies fine)
and then fails in the browser. Hence the test exclusion: CI can't pass
without it, and the comment in `tests.toml` says so.

**Consequence for multi-instance:** `multi_instance = true` works, but each
instance needs its own domain (or a second install will collide at `/`).

## 5. No app login; YunoHost is the only gate

**Decision:** `ldap = "not_relevant"`, `sso = "not_relevant"` in the
manifest; access control is the YunoHost permission on `/`
(`init_main_permission`, default `all_users`).

**Why:** upstream has no authentication at all. Rather than fake an
integration, the permission system is the honest boundary: YunoHost's SSO
session gate sits in front, and anyone who passes it can create jobs and
download results. `doc/ADMIN.md` explicitly warns admins to tighten the
default group, because "all_users" on a shared server is broader than it
sounds. If upstream ever adds auth, revisit this; until then don't add
`ldap = false` either — there is nothing to wire, so "not_relevant" is the
truthful value.

## 6. Deliberately weak systemd sandboxing

**Decision:** `conf/systemd.service` sets only `NoNewPrivileges`,
`PrivateTmp` and a syscall denylist. No `ProtectSystem=strict`, no
`ProtectHome`, no user namespaces blocking.

**Why:** Chromium needs user namespaces and a broad syscall surface; a
standard hardened unit makes the browser crash in ways that look like app
bugs. The comment in the unit file says "minimal on purpose" — keep it
there, or the next maintainer will "fix" it and break scraping. Isolation
instead comes from the dedicated `$app` user and binding to
`127.0.0.1:__PORT__` so the app is never directly reachable; nginx + YunoHost
SSO is the exposed surface.

## 7. nginx tuned for long scrapes and CSV downloads

**Decision:** `conf/nginx.conf` raises `client_max_body_size` to 50M and
`proxy_read_timeout` to 300s above defaults.

**Why:** jobs run for minutes (the proxy must not hang up mid-job) and result
CSVs / keyword-list uploads are large. If users ever report "timeout while a
job runs", raise `proxy_read_timeout` here, don't touch the app.

## 8. ToS disclaimer is part of the package

**Decision:** `doc/PRE_INSTALL.md` warns that headless scraping may violate
Google's Terms of Service and shows the Chromium download size up front.

**Why:** the app is legal to distribute (MIT) but its use sits in a grey
zone. Showing the warning at install time is the responsible placement: it
sets expectations (install takes minutes, downloads hundreds of MB) and puts
the risk decision with the installer. Keep this file if you restructure the
docs.

## Resource numbers, in one place

- `ram.build = 350M`: the browser download/extraction during install.
- `ram.runtime = 1G`: Chromium per active scraping job; cheap when idle,
  this is the headroom, measured off upstream's own guidance for browser
  automation.
- `disk = 1G`: binary (~50 MB) + browsers (~650 MB) + log/CSV headroom.
