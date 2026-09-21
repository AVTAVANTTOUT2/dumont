# ADR-015 — Journal en ajout seul

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D4

## Contexte

Chaque réponse de Jarvis doit être auditable : qu'a-t-il entendu, quel modèle a répondu, quel outil a tourné, qu'a-t-il dit. Le cadrage protégeait le journal par un droit Postgres restreint. Or le rôle applicatif est propriétaire de ses tables (il les crée par migration) et peut se rendre un droit qu'on lui retire ; et le rôle créé par l'image Postgres est superutilisateur.

## Décision

- Table `events`, insertion uniquement, sans `updated_at`.
- Un déclencheur PostgreSQL, posé par une migration, lève une erreur sur `UPDATE`, `DELETE` et `TRUNCATE` de `events`, quel que soit le rôle.
- Chaque étape d'un tour y écrit une ligne : réception de la prise, transcription, provider et modèle, repli, outils, synthèse, fin ou échec ([types](../contracts/data-model.md#events)).
- Une correction est un nouvel événement, jamais une modification.

## Conséquences

- La base grossit ; la purge se fera par archivage, jamais par suppression en place.
- Retirer le déclencheur demande une migration visible en revue : la protection ne tient pas face à un superutilisateur déterminé, elle empêche l'erreur et rend l'intention explicite.
- Un test Pest vérifie qu'un `UPDATE` sur `events` échoue.
