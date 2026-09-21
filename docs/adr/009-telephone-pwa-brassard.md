# ADR-009 — Le téléphone est le brassard : une PWA

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D1

## Contexte

Le brassard physique (carte, écran, micro, molette) demande du matériel et des semaines. Le cadrage ouvrait l'option d'un téléphone en mode kiosque pour la première démonstration. Il faut dès maintenant un tableau de bord pour parler à Jarvis, l'entendre et le commander.

## Décision

- Le brassard du prototype est le téléphone de l'utilisateur, avec une PWA servie par le conteneur app : une page Blade plein écran, du JavaScript en modules ES natifs, Laravel Echo pour le temps réel, un manifest et un service worker pour l'installation sur l'écran d'accueil.
- Pas d'application native, pas de framework front.
- Une seule PWA pour iPhone (Safari) et Android (Chrome) ; les écarts de plateforme sont traités dans [brassard-pwa](../brassard-pwa.md).
- Le service worker ne met en cache que la coquille de l'interface : aucune réponse d'API ni aucun audio n'est servi hors ligne.

## Conséquences

- Aucun store, aucune signature : déployer, c'est recharger la page.
- Limites acceptées : pas d'écoute écran verrouillé, pas de vibration sur iPhone, son à déverrouiller par un geste.
- Le futur brassard physique réutilisera la même page. Filament reste l'outil du poste fixe ([ADR-017](017-filament-administration.md)).

## Alternatives écartées

- Application native : deux plateformes, un store, des semaines de travail pour un prototype.
- Framework front (Vue, React) : du poids et un outillage de plus pour trois écrans.
