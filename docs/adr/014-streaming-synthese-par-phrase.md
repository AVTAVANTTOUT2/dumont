# ADR-014 — Streaming de bout en bout, synthèse par phrase

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D2

## Contexte

La latence est la seule contrainte dure : moins de 3 secondes entre la fin de la prise et le premier son. jarvis-voice ne streame pas : `synthesize()` écrit un WAV complet.

## Décision

- Le LLM répond en flux. L'orchestrateur est un générateur PHP qui découpe le flux en phrases :
  - coupure après `.`, `!`, `?`, `…`, `:` ou un retour à la ligne, suivi d'un espace ou de la fin du flux ;
  - pas de coupure dans un nombre (`3.5`), une abréviation courante (`M.`, `etc.`) ou avant 20 caractères ;
  - coupure forcée à 250 caractères, sur le dernier espace.
- Chaque phrase part au TTS dès qu'elle est complète ; son WAV est annoncé par `tts.chunk`, avec un `seq` croissant, pendant que le modèle continue d'écrire.
- Le téléphone joue les phrases dans l'ordre de `seq`.
- Un tour s'exécute entièrement dans le worker, jamais dans la requête HTTP.
- Le prompt système demande une première phrase courte.

## Conséquences

- Chaque phrase paie sa latence de synthèse complète ; c'est pourquoi la première doit être brève.
- Le TTS est sérialisé (un worker, un GPU) : l'ordre des phrases est garanti côté serveur.
- Le découpeur est la seule logique pure du chemin critique : il a son test unitaire ([ADR-018](018-tests-cibles.md)).
- Pendant qu'une phrase est synthétisée, le flux du LLM s'accumule dans le tampon HTTP et se lit d'un coup au retour : pas de perte, pas de blocage du modèle.
