# Why this package is built this way

A plain-language tour for anyone who might maintain this package later.
Each point ends with a link to the exact lines in this repo that implement it.

## We download a ready-made program instead of compiling it

The app's author already publishes a finished program ("binary") for Linux.
We just download it and pin its checksum so a tampered file gets rejected.
Compiling it ourselves would add a lot of build code that produces the exact
same file.

Downside: the author only builds it for normal 64-bit PCs, so this package
won't run on Raspberry Pi.

Code: [manifest.toml#L44-L47](../manifest.toml#L44-L47) (download + checksum),
[install#L16](../scripts/install#L16) (deploy step),
[manifest.toml#L21](../manifest.toml#L21) (amd64-only)

## The app secretly needs a whole web browser

The app "scrapes" Google Maps by driving a hidden Chrome browser. So during
install it downloads ~650 MB of Chrome. Four choices follow from that:

- **The browser files live in the app's *data* folder, not its program
  folder.** The program folder gets replaced on every update; the data
  folder doesn't. So updating the app doesn't re-download 650 MB every time.
  Code: [install#L30-L32](../scripts/install#L30-L32),
  [_common.sh#L11](../scripts/_common.sh#L11),
  [systemd.service#L11](../conf/systemd.service#L11)
- **Backups skip the browser.** It's 650 MB of something we can always
  re-download. Restoring a backup just downloads it again. Only your scraped
  data (jobs + CSV files) is backed up.
  Code: [backup#L22-L24](../scripts/backup#L22-L24),
  [restore#L35-L37](../scripts/restore#L35-L37)
- **The long list of "apt packages"** in the manifest isn't really this
  app's dependencies — they're small libraries Chrome needs. If Chrome ever
  complains about a missing library, the fix goes in that list.
  Code: [manifest.toml#L63-L64](../manifest.toml#L63-L64)
- **Restore has a permission-fixup block** at the end — YunoHost's restore
  leaves files unreadable by the app unless we fix ownership/modes. Don't
  remove it. Code: [restore#L55-L58](../scripts/restore#L55-L58)

## It must live at the root of a domain (`/`), not `/gmaps`

The app's own web pages assume they're at the root of the domain. Installing
under a sub-path "succeeds" but the site is broken. That's also why each
extra copy of the app needs its own subdomain.

Code: [tests.toml#L4-L8](../tests.toml#L4-L8), [ADMIN.md](ADMIN.md)

## There is no app login — YunoHost is the door

The app itself has zero authentication. Who gets in is decided purely by the
YunoHost "who may visit this URL" permission. The default is all YunoHost
users — tighten it on shared servers, because anyone who gets in can run
scrapes and download results.

Code: [manifest.toml#L24-L25](../manifest.toml#L24-L25)

## We leave the security sandbox deliberately loose

YunoHost packages usually lock their service down hard ("sandboxing"). We
don't, on purpose: Chrome needs freedoms that a strict sandbox removes, and
you get mysterious crashes instead of scrapes. Safety comes from the app
running as its own unprivileged user, only reachable through nginx and the
YunoHost login. Don't "harden" the systemd file without testing a real
scrape afterwards.

Code: [systemd.service#L18-L23](../conf/systemd.service#L18-L23) (the
"minimal on purpose" comment is the trail to follow)

## Small things that are not accidents

- **nginx allows 50 MB uploads and 5-minute responses** — scrapes take
  minutes and CSVs are big. Timeouts during a job → raise it here, not in
  the app. Code: [nginx.conf#L8-L9](../conf/nginx.conf#L8-L9)
- **The install warning is on purpose** — scraping Google Maps may break
  Google's Terms of Service; the installer should know before clicking.
  Code: [PRE_INSTALL.md](PRE_INSTALL.md)
- **RAM/disk numbers in the manifest** are sized for Chrome: ~1 GB while a
  job runs, ~1 GB on disk for program + browser.
  Code: [manifest.toml#L27-L29](../manifest.toml#L27-L29)
