# ADR-006 — PostgreSQL 16 avec pgvector, sans Redis

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D4

## Contexte

Il faut du relationnel, un historique, et plus tard de la recherche vectorielle. Le cadrage ajoutait Redis pour le cache, la file de jobs, les sessions et la présence, soit un conteneur de plus.

## Décision

Une seule base PostgreSQL 16 avec l'extension pgvector (image `pgvector/pgvector:pg16`), dans le conteneur `db`. La file de jobs, le cache et les sessions de Laravel utilisent le pilote `database`. Pas de Redis. Les vecteurs, quand ils arriveront, seront en `vector(1024)`.

## Conséquences

- Une seule sauvegarde, une seule connexion, un seul outil à connaître.
- Le worker interroge la table `jobs` toutes les 200 ms (`--sleep=0.2`) au lieu d'attendre un signal de Redis : environ 100 ms ajoutées en moyenne au démarrage d'un tour, acceptable.
- Si le volume télémétrique explose après le prototype, on ajoutera TimescaleDB, pas un second moteur. Si la file devient un goulot, Redis revient par un nouvel ADR.

## Alternatives écartées

- Redis dès le départ : un quatrième conteneur pour gagner 100 ms.
- Base vectorielle séparée : deux sauvegardes, deux connexions, pour quelques milliers de fragments.
