The app has no login of its own: access is controlled by the YunoHost permission on the app's URL (by default, all YunoHost users). Anyone allowed through SSO can create scraping jobs and download results — grant access carefully.

Scraping options (language, zoom, depth) are set per job in the web UI or API, not in the app settings.

Logs are in `/var/log/$app/$app.log`. The jobs database and CSV results live in `/home/yunohost.app/$app/webdata/`.

If jobs fail with browser errors, the Chromium build can be reinstalled:

```bash
sudo -u $app env PLAYWRIGHT_INSTALL_ONLY=1 \
  PLAYWRIGHT_BROWSERS_PATH=/home/yunohost.app/$app/browsers \
  /var/www/$app/gmaps
```
