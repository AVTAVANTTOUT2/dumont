# ADR-003 — Trois unités d'exécution sur un seul Mac

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D5

## Contexte

Le cadrage prévoyait sept conteneurs (web, reverb, worker, voice, ollama, db, cache) et deux machines. Le prototype tourne sur un seul Mac Apple Silicon (M5, 24 Go), avec une personne au départ. Sept services, c'est sept choses à démarrer, surveiller et comprendre quand ça casse.

## Décision

Trois unités, un dépôt, une commande `make up` :

- **`ai`** — le LLM local, la transcription et la synthèse vocale, ensemble, en natif sur le Mac ([ADR-004](004-unite-ai-native.md)).
- **`app`** — tout le code PHP : web, API, PWA, Reverb, worker, Filament. Un conteneur, trois processus sous supervisord.
- **`db`** — PostgreSQL avec pgvector. Un conteneur.

Un monorepo : `app/`, `ai/`, `docker/`, `docs/` ([structure](../structure.md)).

## Conséquences

- Un seul conteneur applicatif à construire ; web, Reverb et worker partagent l'image et la configuration.
- supervisord relance un worker planté sans toucher au web.
- Pas de haute disponibilité : un nœud, redémarrage automatique, sauvegarde nocturne (M6).
- Séparer web, Reverb et worker en services Compose distincts reste trivial — même image, autre commande — si le besoin apparaît.

## Alternatives écartées

- Les sept conteneurs du cadrage : trop d'opérations pour un Mac et une personne.
- Un conteneur unique avec PHP et Postgres : on mélange cycle de vie du code et des données, et la sauvegarde devient fragile.
