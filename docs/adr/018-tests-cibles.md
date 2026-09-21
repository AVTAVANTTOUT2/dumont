# ADR-018 — Tests limités et ciblés

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D5

## Contexte

Tolérance assumée sur la qualité du code, pas sur les contrats. La voix réelle ne se teste pas en CI : les poids pèsent des gigaoctets, le TTS exige un Mac, et jarvis-voice est privé.

## Décision

Côté Laravel, Pest, cinq catégories seulement :

1. un test de fumée par route HTTP ;
2. un test par outil de Jarvis ;
3. un test par provider LLM, avec un flux SSE simulé (`Http::fake`), repli compris ;
4. le test unitaire du découpeur de phrases ([ADR-014](014-streaming-synthese-par-phrase.md)) ;
5. le test du journal en ajout seul ([ADR-015](015-journal-ajout-seul.md)).

Côté unité ai, pytest marqué `contract` : les routes de la façade avec des moteurs simulés, et la conversion ffmpeg sur de courts fichiers webm, mp4 et wav.

La voix réelle se vérifie par les [mesures](../mesures.md) de chaque jalon, sur le Mac, pas par la CI.

## Conséquences

- CI sous trois minutes.
- Pas de couverture visée, pas de test sur les modèles Eloquent.
- Les régressions d'interface du brassard ne sont pas attrapées automatiquement : la démonstration de fin de jalon et la revue s'en chargent.
