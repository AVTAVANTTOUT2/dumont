# ADR-002 — Reverb pour le temps réel, sur la même origine

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D2

## Contexte

Le serveur doit pousser vers le téléphone la transcription, la réponse qui s'écrit, l'annonce de chaque phrase synthétisée, puis les cartes et les alertes. Le téléphone joint le Mac par une seule URL HTTPS ([ADR-010](010-https-tailscale-serve.md)).

## Décision

Laravel Reverb, dans le conteneur `app`, sous supervisord, en écoute sur 127.0.0.1:8081 à l'intérieur du conteneur. FrankenPHP relaie les chemins `/app/*` et `/apps/*` vers Reverb : le WebSocket partage l'origine HTTPS de l'app. Côté téléphone, Laravel Echo et `pusher-js`. Pas de Node, pas de Redis : Reverb sur un seul nœud n'en a pas besoin.

## Conséquences

- Une seule origine pour la PWA : pas de port WebSocket à publier, pas de CORS, un seul certificat.
- Aucune route Laravel ne commence par `/app` ou `/apps`.
- Reverb monte moins haut en charge qu'une solution dédiée : sans objet pour un téléphone.
- Le sens des messages est fixé par l'[ADR-012](012-audio-http-evenements-ws.md) : le serveur publie, le téléphone écoute.

## Alternatives écartées

- Serveur WebSocket en Node : un langage et un service de plus.
- Server-Sent Events sur la requête d'envoi : simple pour un tour, mais pas pour les alertes hors conversation.
