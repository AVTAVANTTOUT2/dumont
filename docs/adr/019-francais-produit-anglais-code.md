# ADR-019 — Français pour le produit, anglais pour le code

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** tous

## Contexte

Équipe francophone, utilisateur francophone, Jarvis parle français. Le code, lui, vit dans un écosystème anglophone : Laravel, PHP, Python.

## Décision

- **En français :** l'interface, les prompts, les messages destinés à l'utilisateur, la documentation, les ADR, les issues, les PR et le sujet des commits.
- **En anglais :** les noms de classes, méthodes, variables, tables, colonnes, routes, événements et champs JSON, ainsi que le type des commits (`feat`, `fix`…).
- Les commentaires de code suivent la langue du code qu'ils commentent : en anglais.

## Conséquences

- Pas de traduction à gérer au prototype.
- La frontière est nette et se fait respecter en revue : un nom de table en français ou un message utilisateur en anglais est refusé.
