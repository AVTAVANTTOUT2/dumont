# ADR-017 — Filament pour tout écran d'administration

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D4

## Contexte

Il faut visualiser les conversations, relire le journal, réécouter l'audio et révoquer des appareils, sans y passer des semaines. Le cadrage visait Filament 4 ; la version courante est Filament 5.

## Décision

- Filament 5 fournit tout le back-office, à partir du M4 : ressources générées sur les modèles Eloquent (conversations, tours, messages, appareils), `events` en lecture seule, lecteur audio pour les fichiers.
- Aucun écran d'administration écrit à la main.
- Filament s'utilise sur le Mac, dans un navigateur, avec le mot de passe de l'utilisateur. Le téléphone n'a que la PWA.

## Conséquences

- Le back-office ressemble à du Filament, et c'est accepté.
- Seul le brassard a un design propre ([ADR-009](009-telephone-pwa-brassard.md)).
