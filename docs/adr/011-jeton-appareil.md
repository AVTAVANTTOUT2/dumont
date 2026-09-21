# ADR-011 — Authentification par jeton d'appareil

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D4

## Contexte

Un brassard n'a pas de clavier pratique, et une PWA installée garde mal les cookies de session, surtout sur iPhone. Il faut pourtant savoir quel appareil parle, et pouvoir le couper.

## Décision

- Appairage par code à six chiffres, affiché sur le Mac par `php artisan dumont:pair` (puis dans Filament au M4), valable 5 minutes et à usage unique.
- Le téléphone l'échange contre un jeton Sanctum longue durée par `POST /api/v1/devices/pair`. Le jeton appartient à l'appareil (`Device` porte `HasApiTokens`), pas directement à l'utilisateur.
- Le jeton est envoyé en `Authorization: Bearer` sur l'API et sur `/broadcasting/auth`.
- Pas de mot de passe sur le téléphone, pas d'OAuth, pas de SSO.

## Conséquences

- Un téléphone perdu se révoque en supprimant son appareil, ce qui supprime ses jetons.
- Le jeton vit dans le `localStorage` de la PWA : qui a le téléphone déverrouillé a accès à Jarvis. Accepté au prototype.
- Le mot de passe d'utilisateur ne sert qu'à Filament, sur le Mac.
