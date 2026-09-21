# ADR-007 — Providers LLM interchangeables derrière une interface

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D2

## Contexte

DeepSeek en ligne (`deepseek-flash`, déjà utilisé par jarvis-office) est plus fort et rapide. Ollama en local est toujours disponible, même sans réseau. Les deux parlent le dialecte OpenAI.

## Décision

- Une interface PHP `LlmProvider` dans `app/app/Llm` :
  - `chat(array $messages, array $tools = [], bool $stream = true): Generator`
  - `embed(string $text): array`
- Deux implémentations, `DeepSeekProvider` et `OllamaProvider`, avec le même client HTTP de Laravel sur `/v1/chat/completions` en flux SSE.
- Choix par `LLM_DRIVER=deepseek|ollama`. Repli automatique sur Ollama si DeepSeek répond en erreur ou n'a pas envoyé de premier fragment après `LLM_TIMEOUT` secondes (8 au prototype).
- Le provider et le modèle réellement utilisés sont enregistrés à chaque tour et renvoyés dans `assistant.done`.
- Aucun SDK de fournisseur dans le code métier.
- Modèle local proposé : `qwen2.5:7b` (outils pris en charge, bon français, environ 5 Go). Décision finale au M3, après mesure ([#32](https://github.com/AVTAVANTTOUT2/dumont/issues/32)).

## Conséquences

- Une abstraction de plus, assumée.
- Le repli n'agit qu'avant le premier fragment : une réponse commencée n'est jamais rejouée ailleurs.
- Le `qwen2.5:14b` déjà présent sur le Mac (9 Go) laisserait trop peu de mémoire à la voix sur 24 Go ([budget mémoire](../architecture.md#budget-mémoire)).
