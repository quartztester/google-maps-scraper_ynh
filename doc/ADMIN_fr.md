Le scrapeur s'exécute avec le Chromium headless installé pendant l'installation ; vérifiez que le service tourne : `systemctl status $app`.

L'application n'a pas de connexion propre : l'accès est géré par la permission YunoHost sur son URL. Les journaux sont dans `/var/log/$app/$app.log`, les résultats dans `/home/yunohost.app/$app/webdata/`.

En cas d'erreurs de navigateur pendant les jobs, relancez l'installation de Chromium :

```bash
sudo -u $app env PLAYWRIGHT_INSTALL_ONLY=1 \
  PLAYWRIGHT_BROWSERS_PATH=/home/yunohost.app/$app/browsers \
  /var/www/$app/gmaps
```
