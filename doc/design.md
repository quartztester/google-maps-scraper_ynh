# Why this package is built this way

A plain-language tour for anyone who might maintain this package later.
Each point: what we chose, and what would break otherwise.

## We download a ready-made program instead of compiling it

The app's author already publishes a finished program ("binary") for Linux.
We just download it and pin its checksum so a tampered file gets rejected.
Compiling it ourselves would add a lot of build code that produces the exact
same file.

Downside: the author only builds it for normal 64-bit PCs, so this package
won't run on Raspberry Pi.

## The app secretly needs a whole web browser

The app "scrapes" Google Maps by driving a hidden Chrome browser. So during
install it downloads ~650 MB of Chrome. Two choices follow from that:

- **The browser files live in the app's *data* folder, not its program
  folder.** The program folder gets replaced on every update; the data
  folder doesn't. So updating the app doesn't re-download 650 MB every time.
- **Backups skip the browser.** It's 650 MB of something we can always
  re-download. Restoring a backup just downloads it again. Only your scraped
  data (jobs + CSV files) is backed up.
- **The long list of "apt packages"** in the manifest isn't really this
  app's dependencies — they're small libraries Chrome needs. If Chrome ever
  complains about a missing library, the fix goes in that list.

## It must live at the root of a domain (`/`), not `/gmaps`

The app's own web pages assume they're at the root of the domain. Installing
under a sub-path "succeeds" but the site is broken. That's also why each
extra copy of the app needs its own subdomain.

## There is no app login — YunoHost is the door

The app itself has zero authentication. Who gets in is decided purely by the
YunoHost "who may visit this URL" permission. The default is all YunoHost
users — tighten it on shared servers, because anyone who gets in can run
scrapes and download results.

## We leave the security sandbox deliberately loose

YunoHost packages usually lock their service down hard ("sandboxing"). We
don't, on purpose: Chrome needs freedoms that a strict sandbox removes, and
you get mysterious crashes instead of scrapes. Safety comes from the app
running as its own unprivileged user, only reachable through nginx and the
YunoHost login. Don't "harden" the systemd file without testing a real
scrape afterwards.

## Small things that are not accidents

- **nginx allows 50 MB uploads and 5-minute responses** — scrapes take
  minutes and CSVs are big. Timeouts during a job → raise it here, not in
  the app.
- **The scary install warning is on purpose** — scraping Google Maps may
  break Google's Terms of Service; the installer should know before clicking.
- **RAM/disk numbers in the manifest** are sized for Chrome: ~1 GB while a
  job runs, ~1 GB on disk for program + browser.
