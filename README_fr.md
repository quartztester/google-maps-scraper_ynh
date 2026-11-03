<!--
N.B. : Ce README suit la forme produite par le generateur readme_generator de
YunoHost apps_tools ; les sections specifiques au package sont maintenus a la main.
-->

<h1>
  <img src="./logos/gmaps.png" width="32px" alt="Logo de Google Maps Scraper">
  Google Maps Scraper, empaquete pour YunoHost
</h1>

Extracteur de fiches Google Maps avec interface web et API REST

[![Niveau d'integration](https://apps.yunohost.org/badges/level/gmaps)](https://apps.yunohost.org/app/gmaps)
[![Installer Google Maps Scraper avec YunoHost](https://install-app.yunohost.org/install-with-yunohost.svg)](https://install-app.yunohost.org/?app=gmaps)

<div align="center">
<a href="https://apps.yunohost.org/app/gmaps"><img height="100px" src="https://github.com/YunoHost/yunohost-artwork/raw/refs/heads/main/badges/neopossum-badges/badge_more_info_on_the_appstore.svg"/></a>
<a href="https://github.com/quartztester/google-maps-scraper_ynh/issues"><img height="100px" src="https://github.com/YunoHost/yunohost-artwork/raw/refs/heads/main/badges/neopossum-badges/badge_report_an_issue.svg"/></a>
</div>

*[Read this readme in english.](./README.md)*

> *Ce package permet d'installer Google Maps Scraper rapidement et simplement sur un serveur YunoHost. Si vous n'avez pas YunoHost, consultez [le guide](https://yunohost.org/#/install) pour apprendre comment l'installer.*

## Vue d'ensemble

Extracteur de fiches Google Maps avec interface web et API REST.

**Version embarquee :** 1.18.1

## Captures d'ecran

![Interface web](./doc/screenshots/webui.png)

## Avertissement

L'extraction automatisee de Google Maps peut violer les conditions d'utilisation de Google. Utilisez de maniere responsable et a vos risques.

## Documentation et ressources

- Site officiel : https://github.com/gosom/google-maps-scraper
- Depot amont : https://github.com/gosom/google-maps-scraper
- Documentation d'administration du package : [doc/ADMIN_fr.md](./doc/ADMIN_fr.md)
- Forums communautaires YunoHost : https://forum.yunohost.org

## Limitations

L'interface web et l'API utilisent des URLs absolues : l'app doit etre installee a la racine de son domaine (chemin `/`). L'extraction necessite le Chromium headless integre (quelques centaines de Mo, installes automatiquement).

## Panneau de configuration

Ce package n'embarque pas de panneau de configuration : les options d'extraction se reglent par tache dans l'interface web/l'API, et le binaire amont ne lit aucun fichier de configuration serveur.

## Informations developpeur

- Depot du package : https://github.com/quartztester/google-maps-scraper_ynh
- Signaler un bug dans ce package : https://github.com/quartztester/google-maps-scraper_ynh/issues

Reportez-vous a [la documentation d'empaquetage](https://doc.yunohost.org/en/dev/packaging/).
