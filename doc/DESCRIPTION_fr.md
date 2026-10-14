Google Maps Scraper extrait les données des fiches Google Maps : nom, adresse, téléphone, site web, e-mails, notes, avis, coordonnées, etc., en CSV ou via une API REST.

L'interface web permet de créer des jobs d'extraction, de suivre leur progression et de télécharger les résultats. Une API REST documentée (OpenAPI) est disponible sous `/api/v1/`.

L'extraction utilise un Chromium headless installé sur le serveur. Les résultats apparaissent après au moins ~3 minutes par job.

**Important :** l'extraction automatisée de Google Maps peut violer les conditions d'utilisation de Google. Utilisez cette application de manière responsable et à vos risques.
