# ADR-008 — Les embeddings ne sortent jamais du Mac

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D2

## Contexte

La documentation qui sera indexée après le prototype peut être sensible. Vectoriser chez un fournisseur en ligne revient à lui envoyer tout le corpus.

## Décision

La vectorisation passe toujours par Ollama en local, avec `bge-m3` (1024 dimensions, multilingue), même quand le chat passe par DeepSeek. `LlmProvider::embed()` appelle toujours Ollama, quel que soit `LLM_DRIVER`.

## Conséquences

- Ollama est une dépendance permanente, y compris en mode en ligne.
- Le corpus ne quitte pas le Mac ; seules les questions et les fragments retenus partent chez DeepSeek quand il est actif.
- Sans usage avant le RAG, après le prototype. La décision est prise maintenant pour ne pas se fermer de porte.
