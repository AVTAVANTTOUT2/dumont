# Dumont — consignes pour les agents de code

Dumont est un assistant vocal Jarvis : une PWA sur le téléphone, Laravel 13 dans Docker, la voix et le LLM local en natif sur un Mac Apple Silicon. Avant toute tâche, lire :

1. l'issue traitée et les ADR qu'elle cite ;
2. [docs/architecture.md](docs/architecture.md) et [docs/structure.md](docs/structure.md) ;
3. le contrat concerné dans [docs/contracts/](docs/contracts/README.md).

## Règles non négociables

- Les ADR priment sur l'issue, sur les préférences et sur les habitudes du framework. Si la tâche contredit un ADR, s'arrêter et le signaler : la décision change d'abord, dans une PR `adr`.
- Une issue, une branche `type/dN-slug`, une PR de 400 lignes au plus, remplie avec le modèle. Ne jamais pousser sur `main`, ne jamais fusionner soi-même.
- Toute route, tout événement temps réel, toute table et tout changement du service ai met à jour `docs/contracts/` dans la même PR, étiquetée `contract`.
- Ne jamais modifier `ai/jarvis-voice/` (sous-module). Ne jamais copier dans ce dépôt la voix `jarvis-fr`, son WAV de référence ou des poids de modèle.
- Les services de l'unité ai écoutent sur `127.0.0.1` uniquement. Rien n'est exposé hors du tailnet ; pas de Tailscale Funnel.
- Aucun secret dans Git : `.env` est ignoré, seul `.env.example` est versionné.
- Python n'existe que dans `ai/`. Tout le chemin d'une requête utilisateur est en PHP.
- Pas de framework d'agents ni de framework front ([ADR-016](docs/adr/016-outils-classes-php.md), [ADR-009](docs/adr/009-telephone-pwa-brassard.md)). Toute nouvelle dépendance est citée dans la PR.
- Français pour la documentation, l'interface, les prompts et les sujets de commit. Anglais pour les classes, tables, colonnes, routes et variables.

## Où est quoi

| Chemin | Contenu |
|---|---|
| `app/` | Laravel 13, conteneur `app` (arrive au M0) |
| `ai/` | unité ai en natif : façade voix FastAPI, worker TTS, sous-module jarvis-voice (arrive au M0) |
| `docker/` | images et configuration des conteneurs (arrive au M0) |
| `docs/` | vision, architecture, structure, ADR, contrats, feuille de route, mesures |

## Commandes

Disponibles à partir du M0 : `make setup`, `make models`, `make up`, `make down`, `make ai`, `make phone`, `make logs`, `make test`.

## Commits et PR

`type(dN): sujet à l'impératif en français`, par exemple `feat(d3): ajouter la route POST /tts`. Le détail est dans [CONTRIBUTING.md](CONTRIBUTING.md).
