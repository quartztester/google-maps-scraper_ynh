<!--
N.B.: This README follows the shape of the YunoHost apps_tools readme_generator
output; hand-maintained sections carry the package's own disclaimers and limits.
-->

<h1>
  <img src="./logos/gmaps.png" width="32px" alt="Logo of Google Maps Scraper">
  Google Maps Scraper, packaged for YunoHost
</h1>

Scrape Google Maps business listings with a web UI and REST API

[![Integration level](https://apps.yunohost.org/badges/level/gmaps)](https://apps.yunohost.org/app/gmaps)
[![Install Google Maps Scraper with YunoHost](https://install-app.yunohost.org/install-with-yunohost.svg)](https://install-app.yunohost.org/?app=gmaps)

<div align="center">
<a href="https://apps.yunohost.org/app/gmaps"><img height="100px" src="https://github.com/YunoHost/yunohost-artwork/raw/refs/heads/main/badges/neopossum-badges/badge_more_info_on_the_appstore.svg"/></a>
<a href="https://github.com/quartztester/google-maps-scraper_ynh/issues"><img height="100px" src="https://github.com/YunoHost/yunohost-artwork/raw/refs/heads/main/badges/neopossum-badges/badge_report_an_issue.svg"/></a>
</div>

*[Lire ce readme en français.](./README_fr.md)*

> *This package allows you to install [Google Maps Scraper](https://github.com/gosom/google-maps-scraper) quickly and simply on a YunoHost server.*
> *If you don't have YunoHost, please consult [the guide](https://yunohost.org/#/install) to learn how to install it.*

## Overview

Scrape Google Maps business listings with a web UI and REST API.

**Shipped version:** 1.18.1

## Screenshots

![Web UI](./doc/screenshots/webui.png)

## Disclaimer

Automated scraping of Google Maps may violate Google's Terms of Service. Use responsibly, at modest volume, and at your own risk.

## Documentation and resources

- Official app website: https://github.com/gosom/google-maps-scraper
- Upstream repository: https://github.com/gosom/google-maps-scraper
- Package admin documentation: [doc/ADMIN.md](./doc/ADMIN.md)
- YunoHost community forums: https://forum.yunohost.org

## Limitations

The web UI and API hardcode absolute URLs, so this app must be installed at the root of its domain (path `/`). Scraping requires the bundled headless Chromium (a few hundred MB, installed automatically).

## Config panel

This package does not ship a config panel: scraping options are set per job in the web UI/API, and the upstream binary takes no server-side config file.

## Developer info

- Package repository: https://github.com/quartztester/google-maps-scraper_ynh
- Report a bug in this package: https://github.com/quartztester/google-maps-scraper_ynh/issues

Please read [the YunoHost doc on packaging](https://doc.yunohost.org/en/dev/packaging/), and in particular these docs about [app packaging](https://doc.yunohost.org/en/dev/packaging/apps).

You are advised to clone the repo and install the app with `yunohost app install https://github.com/quartztester/google-maps-scraper_ynh`.

### How to submit a PR

When submitting a pull request, please make sure that your branch is up to date with the main branch and that you tested the install/upgrade cycle.

#### Notable software components this app uses:

* [Go](https://endoflife.date/go): programming language the binary is built with

#### Best Practices

Please read [the guidelines](https://doc.yunohost.org/en/dev/packaging/best_practices) before contributing.
