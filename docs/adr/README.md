# Décisions d'architecture (ADR)

Une décision structurante par fichier, au format du [modèle](000-modele.md). Elles sont **figées pendant un jalon** et ne se rediscutent qu'entre deux jalons, en revue d'ADR. Une PR qui contredit un ADR est refusée sans débat : on modifie l'ADR d'abord, dans une PR dédiée étiquetée `adr`, sans code.

Ce jeu remplace les quinze ADR du document de cadrage du 21 septembre 2026. Les écarts sont listés dans l'[architecture](../architecture.md#écarts-avec-le-document-de-cadrage).

| N° | Décision | Domaine | Statut |
|---|---|---|---|
| [001](001-laravel-socle-unique.md) | Laravel 13 sur FrankenPHP comme socle unique | D5 | Acceptée |
| [002](002-reverb-temps-reel.md) | Reverb pour le temps réel, sur la même origine | D2 | Acceptée |
| [003](003-trois-unites-un-mac.md) | Trois unités d'exécution sur un seul Mac | D5 | Acceptée |
| [004](004-unite-ai-native.md) | L'unité ai tourne en natif sur le Mac | D3 | Acceptée |
| [005](005-jarvis-voice-sous-module.md) | jarvis-voice intégré tel quel, poids hors de Git | D3 | Acceptée |
| [006](006-postgres-pgvector-sans-redis.md) | PostgreSQL 16 avec pgvector, sans Redis | D4 | Acceptée |
| [007](007-providers-llm-interchangeables.md) | Providers LLM interchangeables derrière une interface | D2 | Acceptée |
| [008](008-embeddings-locaux.md) | Les embeddings ne sortent jamais du Mac | D2 | Acceptée |
| [009](009-telephone-pwa-brassard.md) | Le téléphone est le brassard : une PWA | D1 | Acceptée |
| [010](010-https-tailscale-serve.md) | HTTPS et accès distant par Tailscale Serve | D5 | Acceptée |
| [011](011-jeton-appareil.md) | Authentification par jeton d'appareil | D4 | Acceptée |
| [012](012-audio-http-evenements-ws.md) | L'audio monte en HTTP, les événements descendent en WebSocket | D2 | Acceptée |
| [013](013-push-to-talk-dabord.md) | Push-to-talk d'abord, mains libres ensuite | D1 | Acceptée |
| [014](014-streaming-synthese-par-phrase.md) | Streaming de bout en bout, synthèse par phrase | D2 | Acceptée |
| [015](015-journal-ajout-seul.md) | Journal en ajout seul | D4 | Acceptée |
| [016](016-outils-classes-php.md) | Les outils de Jarvis sont des classes PHP | D2 | Acceptée |
| [017](017-filament-administration.md) | Filament pour tout écran d'administration | D4 | Acceptée |
| [018](018-tests-cibles.md) | Tests limités et ciblés | D5 | Acceptée |
| [019](019-francais-produit-anglais-code.md) | Français pour le produit, anglais pour le code | tous | Acceptée |

## Écrire ou changer un ADR

1. Copier le [modèle](000-modele.md) sous le numéro suivant, titre court en kebab-case.
2. Ouvrir une PR étiquetée `adr`, branche `docs/<domaine>-adr-<slug>`, qui ne touche que `docs/adr/` et, si besoin, la documentation qui en découle.
3. Pour remplacer une décision : nouvel ADR, et l'ancien passe au statut « Remplacée par ADR-NNN ». On ne réécrit pas l'histoire d'un ADR accepté.
4. Ajouter la ligne dans la table ci-dessus.
