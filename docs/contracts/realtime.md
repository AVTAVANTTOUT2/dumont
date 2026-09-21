# Événements temps réel

Laravel Reverb, protocole Pusher, sur la même origine que l'app (`wss://<mac>.<tailnet>.ts.net/app/<clé>`). Le serveur publie, le téléphone écoute : aucun message n'est envoyé du téléphone vers le serveur par WebSocket ([ADR-012](../adr/012-audio-http-evenements-ws.md)). Propriétaire : D2.

## Canaux

| Canal | Type | Autorisé si | Jalon |
|---|---|---|---|
| `conversation.{id}` | privé | la conversation appartient à l'utilisateur de l'appareil | M1 |
| `device.{id}` | privé | c'est l'appareil authentifié | M5 |

Autorisation par `POST /broadcasting/auth` avec le jeton d'appareil en `Authorization: Bearer`. Côté client, avec Laravel Echo :

```js
Echo.private(`conversation.${id}`).listen('.transcript.final', (e) => { /* … */ });
```

Le point devant le nom est obligatoire : les événements portent un nom explicite (`broadcastAs`), pas le nom de leur classe PHP.

## Règles communes

- Chaque charge utile porte `turn_id` (sauf `alert`) et `at`, horodatage ISO 8601.
- Une charge utile pèse moins de 8 Ko. Jamais d'audio dans un événement : l'audio se télécharge par son URL.
- Les événements d'un tour sont émis par un seul worker, dans l'ordre ; le client ne doit pas pour autant supposer l'ordre de fin des téléchargements audio (voir `seq`).
- Ajouter un événement ou un champ facultatif est libre, en l'écrivant ici. Renommer, retirer ou changer le type d'un champ exige l'étiquette `contract`.

## Événements

| Événement | Canal | Jalon | Quand |
|---|---|---|---|
| `transcript.final` | conversation | M2 | la prise est transcrite |
| `assistant.delta` | conversation | M3 | un morceau de réponse arrive |
| `assistant.done` | conversation | M3 | la réponse texte est complète |
| `tts.chunk` | conversation | M1 | une phrase est synthétisée et prête à jouer |
| `turn.done` | conversation | M1 | le tour est fini, synthèse comprise |
| `turn.failed` | conversation | M1 | le tour s'arrête sur une erreur |
| `tool.started` | conversation | M5 | un outil démarre |
| `tool.done` | conversation | M5 | un outil a fini |
| `ui.render` | conversation | M5 | afficher une carte sur le tableau de bord |
| `haptic` | conversation | M5 | jouer un motif de vibration |
| `alert` | device | M5 | alerte hors conversation, par exemple un rappel |

### transcript.final

```json
{ "turn_id": "01J8Z5…", "text": "Quelle heure est-il ?", "duration_ms": 2140, "at": "…" }
```

`text` vide : rien n'a été entendu ; suit directement `turn.done`.

### assistant.delta

```json
{ "turn_id": "01J8Z5…", "text": "Il est quatorze heures", "at": "…" }
```

Les fragments du LLM sont regroupés avant émission, par phrase ou toutes les 200 ms, pour ne pas saturer Reverb. Le client les concatène.

### assistant.done

```json
{ "turn_id": "01J8Z5…", "text": "Il est quatorze heures cinquante-deux.", "provider": "deepseek", "model": "deepseek-flash", "fallback": false, "ms": 1480, "at": "…" }
```

`fallback` vaut `true` quand Ollama a pris le relais de DeepSeek.

### tts.chunk

```json
{ "turn_id": "01J8Z5…", "seq": 0, "url": "/api/v1/audio/01J8Z7…", "text": "Il est quatorze heures cinquante-deux.", "duration_ms": 2310, "at": "…" }
```

- `seq` part de 0 et croît de 1 à chaque phrase du tour. Le client joue dans l'ordre de `seq`, quel que soit l'ordre de fin des téléchargements.
- `url` est relative à l'origine de l'app et se télécharge avec le jeton.

### turn.done

```json
{ "turn_id": "01J8Z5…", "tts_chunks": 2, "total_ms": 2950, "at": "…" }
```

`tts_chunks` dit au client combien de phrases attendre avant de revenir au repos.

### turn.failed

```json
{ "turn_id": "01J8Z5…", "stage": "tts", "message": "La voix de Jarvis est indisponible, réponse en texte seulement.", "retryable": true, "at": "…" }
```

- `stage` : `upload`, `stt`, `llm`, `tts` ou `tool`.
- `message` : en français, affichable tel quel.
- Un échec en `tts` après `assistant.done` laisse la réponse texte valable : c'est le mode dégradé.

### tool.started et tool.done

```json
{ "turn_id": "01J8Z5…", "tool": "create_note", "label": "Création de la note", "at": "…" }
```

```json
{ "turn_id": "01J8Z5…", "tool": "create_note", "ok": true, "ms": 42, "at": "…" }
```

### ui.render

```json
{ "turn_id": "01J8Z5…", "component": "note", "props": { "title": "Note ajoutée", "body": "La porte du garage grince" }, "at": "…" }
```

`component` : `note`, `status`, `checklist` ou `reminder`. Le brassard ignore un composant qu'il ne connaît pas.

### haptic

```json
{ "turn_id": "01J8Z5…", "pattern": "confirm", "at": "…" }
```

`pattern` : `confirm`, `alert` ou `error`. Ignoré sur iPhone, où la vibration n'existe pas pour le web.

### alert

```json
{ "level": "info", "message": "Rappel : sortir le linge.", "source": "reminder", "at": "…" }
```

`level` : `info`, `warning` ou `critical`.

## Réservés et retirés

- `transcript.partial` est réservé au mode mains libres avec transcription en continu ; pas avant le M6, et seulement si un ADR rétablit l'audio en flux.
- `audio.chunk` du cadrage est retiré : l'audio monte par `POST /api/v1/turns`.
