# ADR-010 — HTTPS et accès distant par Tailscale Serve

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D5

## Contexte

Le micro (`getUserMedia`), le service worker et l'installation d'une PWA exigent un contexte sécurisé : HTTPS avec un certificat reconnu par le téléphone, dès qu'on sort de `localhost`. Le téléphone doit joindre le Mac à la maison comme en déplacement. Tailscale est déjà installé et connecté sur le Mac.

## Décision

- Tailscale Serve publie `http://127.0.0.1:8080` à l'adresse `https://<mac>.<tailnet>.ts.net`, avec un certificat valide, visible uniquement dans le tailnet.
- Le téléphone a l'application Tailscale et rejoint le même tailnet.
- `make phone` active la publication, `make phone-off` la retire.
- `APP_URL` vaut l'adresse `ts.net`.
- Rien n'est exposé sur Internet ni sur le réseau local : app et db publient leurs ports sur 127.0.0.1 seulement.

## Conséquences

- Le téléphone doit être connecté au tailnet pour joindre Jarvis.
- Une seule adresse, identique à la maison et dehors.
- Tailscale Funnel, qui exposerait l'app sur Internet, est interdit sans nouvel ADR.

## Alternatives écartées

- mkcert et autorité locale : il faut installer et approuver l'autorité sur le téléphone, laborieux sur iPhone, et rien ne marche hors du réseau local.
- Tunnel public ou nom de domaine : expose l'app sur Internet pour un besoin personnel.
