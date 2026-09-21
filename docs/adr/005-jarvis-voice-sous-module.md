# ADR-005 — jarvis-voice intégré tel quel, poids hors de Git

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D3

## Contexte

jarvis-voice fige les modèles (faster-whisper-large-v3-turbo, Qwen3-TTS-12Hz-0.6B-Base-6bit, Silero 6.2.1), leurs réglages de production et le profil de voix `jarvis-fr`, dans un dépôt privé. Il expose deux classes Python, `Transcriber` et `Speaker`, pas un service. Ses poids pèsent environ 3,2 Go. Le profil `jarvis-fr` est à usage privé et non redistribuable. Ses chemins d'actifs sont relatifs à sa propre racine.

## Décision

- jarvis-voice est un sous-module Git en `ai/jarvis-voice`, épinglé sur un commit.
- Il est installé en mode éditable dans les deux environnements de l'unité ai : `.venv-stt` (Python 3.12, extra `stt`) et `.venv-tts` (Python 3.14, extra `tts`). Ses actifs restent dans son répertoire.
- Dumont ne modifie jamais son code. Un changement passe par une PR sur jarvis-voice, puis par une montée de commit du sous-module dans Dumont.
- Les poids ne sont pas dans Git : `make models` lance `python -m jarvis_voice fetch` puis `check`, qui vérifie les empreintes SHA-256. Les modèles Ollama se récupèrent par `ollama pull`, leurs noms fixés dans `.env`.

## Conséquences

- Dumont reste un dépôt privé tant qu'il référence jarvis-voice.
- La voix `jarvis-fr` n'est jamais copiée dans ce dépôt ni dans un artefact. Une démonstration hors équipe attend la décision [#34](https://github.com/AVTAVANTTOUT2/dumont/issues/34).
- Le premier démarrage est long : environ 3,2 Go de poids voix et 5 Go de modèles Ollama.
- La CI n'a pas accès au sous-module : les tests de contrat de la façade simulent les moteurs ([ADR-018](018-tests-cibles.md)).

## Alternatives écartées

- Copier le code de jarvis-voice dans `ai/` : deux versions divergent en un mois.
- Dépendance pip vers le dépôt Git : ses actifs, relatifs à sa racine, ne suivent pas une installation non éditable.
