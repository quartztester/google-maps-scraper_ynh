1|# Why this package is built this way
2|
3|A plain-language tour for anyone who might maintain this package later.
4|Each point ends with a link to the exact lines in this repo that implement it.
5|
6|## We download a ready-made program instead of compiling it
7|
8|The app's author already publishes a finished program for Linux.
9|We just download it and pin its checksum so a tampered file gets rejected.
10|Compiling it ourselves would add a lot of build code that produces the exact
11|same file.
12|
13|Downside: the author only builds it for normal 64-bit PCs, so this package
14|won't run on Raspberry Pi.
15|
16|Code: [manifest.toml#L44-L47](../manifest.toml#L44-L47) (download + checksum),
17|[install#L16](../scripts/install#L16) (deploy step),
18|[manifest.toml#L21](../manifest.toml#L21) (amd64-only)
19|
20|## The app secretly needs a whole web browser
21|
22|The app "scrapes" Google Maps by driving a hidden Chrome browser. So during
23|install it downloads ~650 MB of Chrome. Four choices follow from that:
24|
25|- **The browser files live in the app's *data* folder, not its program
26|  folder.** The program folder gets replaced on every update; the data
27|  folder doesn't. So updating the app doesn't re-download 650 MB every time.
28|  Code: [install#L30-L32](../scripts/install#L30-L32),
29|  [_common.sh#L11](../scripts/_common.sh#L11),
30|  [systemd.service#L11](../conf/systemd.service#L11)
31|- **Backups skip the browser.** It's 650 MB of something we can always
32|  re-download. Restoring a backup just downloads it again. Only your scraped
33|  data (jobs + CSV files) is backed up.
34|  Code: [backup#L22-L24](../scripts/backup#L22-L24),
35|  [restore#L35-L37](../scripts/restore#L35-L37)
36|- **The long list of "apt packages"** in the manifest isn't really this
37|  app's dependencies — they're small libraries Chrome needs. If Chrome ever
38|  complains about a missing library, the fix goes in that list.
39|  Code: [manifest.toml#L63-L64](../manifest.toml#L63-L64)
40|- **Restore has a permission-fixup block** at the end — YunoHost's restore
41|  leaves files unreadable by the app unless we fix ownership/modes. Don't
42|  remove it. Code: [restore#L55-L58](../scripts/restore#L55-L58)
43|
44|## It must live at the root of a domain (`/`), not `/gmaps`
45|
46|The app's own web pages assume they're at the root of the domain. Installing
47|under a sub-path "succeeds" but the site is broken. That's also why each
48|extra copy of the app needs its own subdomain.
49|
50|Code: [tests.toml#L4-L8](../tests.toml#L4-L8), [ADMIN.md](ADMIN.md)
51|
52|## There is no app login — YunoHost is the door
53|
54|The app itself has zero authentication. Who gets in is decided purely by the
55|YunoHost "who may visit this URL" permission. The default is all YunoHost
56|users — tighten it on shared servers, because anyone who gets in can run
57|scrapes and download results.
58|
59|Code: [manifest.toml#L24-L25](../manifest.toml#L24-L25)
60|
61|## We leave the security sandbox deliberately loose
62|
63|YunoHost packages usually lock their service down hard. We
64|don't, on purpose: Chrome needs freedoms that a strict sandbox removes, and
65|you get mysterious crashes instead of scrapes. Safety comes from the app
66|running as its own unprivileged user, only reachable through nginx and the
67|YunoHost login. Don't "harden" the systemd file without testing a real
68|scrape afterwards.
69|
70|Code: [systemd.service#L18-L23](../conf/systemd.service#L18-L23) (the
71|"minimal on purpose" comment is the trail to follow)
72|
73|## Small things that are not accidents
74|
75|- **nginx allows 50 MB uploads and 5-minute responses** — scrapes take
76|  minutes and CSVs are big. Timeouts during a job → raise it here, not in
77|  the app. Code: [nginx.conf#L8-L9](../conf/nginx.conf#L8-L9)
78|- **The install warning is on purpose** — scraping Google Maps may break
79|  Google's Terms of Service; the installer should know before clicking.
80|  Code: [PRE_INSTALL.md](PRE_INSTALL.md)
81|- **RAM/disk numbers in the manifest** are sized for Chrome: ~1 GB while a
82|  job runs, ~1 GB on disk for program + browser.
83|  Code: [manifest.toml#L27-L29](../manifest.toml#L27-L29)
84|