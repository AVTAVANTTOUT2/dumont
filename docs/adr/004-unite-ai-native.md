# ADR-004 — L'unité ai tourne en natif sur le Mac

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D3

## Contexte

On voulait le LLM, le STT et le TTS dans un même conteneur, pour la simplicité. Or jarvis-voice synthétise avec Qwen3-TTS via mlx-audio, qui exige Metal sur Apple Silicon, et Docker Desktop sur macOS exécute une machine virtuelle Linux sans accès au GPU. Dans un conteneur, la voix Jarvis ne tourne pas du tout, et Ollama tourne sur CPU, trois à cinq fois plus lentement. Le STT (faster-whisper) tourne sur CPU dans les deux cas.

## Décision

`ai` est une unité logique : un ensemble lancé et arrêté d'un bloc, pas un conteneur. Elle regroupe deux processus natifs :

- **Ollama** (`ollama serve`) sur 127.0.0.1:11434 : chat et embeddings, API compatible OpenAI, accélération Metal.
- **La façade voix `dumont-ai`** (FastAPI) sur 127.0.0.1:8100 : `POST /stt`, `POST /tts`, `GET /health`. Elle tourne en Python 3.12 avec le STT en mémoire, et pilote un worker TTS persistant en Python 3.14 — les deux interpréteurs qu'impose jarvis-voice — par JSON ligne à ligne sur l'entrée et la sortie standard.

`make ai` démarre l'ensemble au début ; un LaunchAgent le démarre à l'ouverture de session à partir du M6. Les deux services écoutent uniquement en boucle locale ; le conteneur app les joint par `host.docker.internal`. Contrat : [ai-service](../contracts/ai-service.md).

## Conséquences

- La voix Jarvis et Ollama sur GPU sont conservés.
- L'unité ai n'est pas reproductible par Docker : `make setup` vérifie uv, ffmpeg et Ollama sur le Mac, et `make models` récupère les poids.
- La façade n'a pas d'authentification : sa sécurité repose sur l'écoute en boucle locale. Elle ne s'expose jamais au réseau.
- Pour un futur serveur Linux, la façade pourra être conteneurisée avec un autre moteur TTS, sans changer son contrat.

## Alternatives écartées

- Tout dans Docker avec une voix générique (Kokoro, Piper) : on perd la voix Jarvis clonée, et Ollama devient lent.
- Conteneur ai avec Ollama et STT, plus une passerelle TTS native : quatre unités pour le même résultat.
