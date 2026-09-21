# ADR-016 — Les outils de Jarvis sont des classes PHP

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D2

## Contexte

Pour que Jarvis obéisse, le LLM doit pouvoir appeler des fonctions. La tentation est d'installer une bibliothèque d'agents, avec ses abstractions et ses montées de version.

## Décision

- Une classe par outil dans `app/app/Tools`, découverte automatiquement.
- Chaque outil déclare un nom, une description en français, un schéma JSON statique de ses arguments, et une méthode `handle()` qui rend un résultat sérialisable.
- Les outils sont exposés aux deux providers au format OpenAI ([ADR-007](007-providers-llm-interchangeables.md)).
- Un seul tour d'outils par requête au prototype : le LLM appelle, on exécute, le LLM formule la réponse.
- Chaque appel est enregistré dans `tool_calls` et dans `events`, et annoncé par `tool.started` et `tool.done`.
- Un outil qui agit hors de Dumont (envoyer un message, piloter un appareil) demande une confirmation vocale explicite avant d'agir. La plateforme propose, l'humain décide.

## Conséquences

- Une trentaine de lignes par outil, aucune dépendance, aucune montée de version subie.
- Pas de chaînage automatique d'outils au prototype.
- Chaque outil a son test Pest ([ADR-018](018-tests-cibles.md)).
