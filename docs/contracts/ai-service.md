# Contrat du service ai

L'unité ai expose deux services, tous deux en boucle locale sur le Mac. Le conteneur app les joint par `host.docker.internal`. Aucune authentification : ils ne doivent jamais écouter ailleurs que sur `127.0.0.1` ([ADR-004](../adr/004-unite-ai-native.md)). Propriétaire : D3.

| Service | Sur le Mac | Depuis app |
|---|---|---|
| Façade voix `dumont-ai` | `127.0.0.1:8100` | `http://host.docker.internal:8100` |
| Ollama | `127.0.0.1:11434` | `http://host.docker.internal:11434` |

La façade ne connaît ni la base, ni les utilisateurs, ni le métier : elle transforme de l'audio en texte et du texte en audio.

## Façade voix

| Méthode | Route | Rôle | Jalon |
|---|---|---|---|
| GET | `/health` | état des moteurs | M0 |
| POST | `/tts` | texte → WAV | M1 |
| POST | `/stt` | audio → texte | M2 |

### GET /health

Répond `200` dès que la façade tourne, même si un moteur manque : l'état est dans le corps.

```json
{
  "status": "ok",
  "stt": { "ready": true, "model": "faster-whisper-large-v3-turbo", "last_ms": 812 },
  "tts": { "ready": true, "model": "Qwen3-TTS-12Hz-0.6B-Base-6bit", "voice": "jarvis-fr", "worker_pid": 4242, "last_ms": 1130 },
  "ffmpeg": true,
  "jarvis_voice": "a1b2c3d"
}
```

- `status` vaut `ok` si les deux moteurs sont prêts, `degraded` sinon.
- `last_ms` vaut `null` avant le premier appel.
- `jarvis_voice` est le commit court du sous-module.

### POST /tts

Entrée `application/json` :

```json
{ "text": "Bonjour, je suis Jarvis." }
```

- `text` : 1 à 500 caractères. Une phrase, deux au plus : le découpage est fait par l'orchestrateur ([ADR-014](../adr/014-streaming-synthese-par-phrase.md)).

Réponse `200`, `audio/wav` : PCM 16 bits, mono, 24 kHz, voix `jarvis-fr`. En-têtes :

| En-tête | Contenu |
|---|---|
| `X-Duration-Ms` | durée de l'audio produit |
| `X-Processing-Ms` | temps de synthèse |

Les requêtes sont traitées une à une, dans l'ordre d'arrivée : un worker TTS, un GPU.

| Code | Cas |
|---|---|
| 400 | texte absent, vide ou trop long |
| 503 | worker TTS indisponible ; il est relancé automatiquement, l'appelant peut réessayer une fois |
| 504 | synthèse de plus de 30 s |

### POST /stt

Entrée `multipart/form-data` :

| Champ | Type | Obligatoire | Note |
|---|---|---|---|
| `audio` | fichier | oui | tout format lu par ffmpeg : `webm/opus` (Android), `mp4/aac` (iPhone), `wav` |
| `language` | texte | non | `fr` par défaut |

Traitement : ffmpeg convertit en WAV PCM 16 bits, 16 kHz, mono, puis `Transcriber.transcribe_wav` de jarvis-voice (large-v3-turbo, CPU, `beam_size=1`).

Réponse `200` :

```json
{ "text": "Quelle heure est-il ?", "language": "fr", "duration_ms": 2140, "processing_ms": 690 }
```

Un `text` vide est une réponse valide : la prise ne contenait que du silence.

| Code | Cas |
|---|---|
| 400 | fichier absent ou illisible par ffmpeg |
| 413 | audio de plus de 60 s, la limite de jarvis-voice |
| 503 | moteur STT non chargé |

## Ollama

L'API d'Ollama telle quelle, appelée par app dans son dialecte OpenAI ([ADR-007](../adr/007-providers-llm-interchangeables.md)) :

| Méthode | Route | Usage | Modèle |
|---|---|---|---|
| POST | `/v1/chat/completions` | chat en flux (`stream: true`), outils | `OLLAMA_CHAT_MODEL` |
| POST | `/v1/embeddings` | vectorisation, après le prototype | `OLLAMA_EMBED_MODEL`, 1024 dimensions |
| GET | `/api/version` | santé, lue par `GET /api/v1/health` | — |

## Protocole interne façade ↔ worker TTS

Détail d'implémentation, pas un contrat pour app ; documenté ici parce qu'il traverse deux interpréteurs. Une ligne JSON par message sur l'entrée et la sortie standard du worker (Python 3.14, `.venv-tts`).

```text
← {"ready": true, "model": "Qwen3-TTS-12Hz-0.6B-Base-6bit", "sample_rate": 24000}
→ {"id": "7f3c", "text": "Bonjour.", "output": "/tmp/dumont-ai/7f3c.wav"}
← {"id": "7f3c", "ok": true, "duration_ms": 830, "processing_ms": 610}
← {"id": "7f3c", "ok": false, "error": "empty synthesis"}
```

- Le worker charge `Speaker` une fois, puis annonce `ready`.
- Les journaux du worker vont sur sa sortie d'erreur, jamais sur sa sortie standard.
- Si le worker meurt, la façade répond `503` aux requêtes en cours et le relance.

## Mesures attendues

Latence TTS au M1, latence STT au M2, reportées dans [mesures.md](../mesures.md).
