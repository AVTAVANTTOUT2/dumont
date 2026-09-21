# API REST — /api/v1

Consommée par la PWA du brassard. Réponses en JSON, erreurs au format Laravel (`message`, `errors`). Noms de routes et de champs en anglais, messages destinés à l'utilisateur en français ([ADR-019](../adr/019-francais-produit-anglais-code.md)). Propriétaire : D2, et D4 pour l'appairage.

**Authentification** : jeton d'appareil Sanctum en `Authorization: Bearer <jeton>`, sauf mention contraire ([ADR-011](../adr/011-jeton-appareil.md)). Sans jeton valide : `401`. Ressource d'un autre utilisateur : `404`.

**Identifiants** : les conversations, tours et fichiers audio ont des ULID.

## Routes

| Méthode | Route | Auth | Rôle | Jalon |
|---|---|---|---|---|
| GET | `/health` | non | état de app, db, ai et du LLM | M0 |
| POST | `/devices/pair` | non | échanger un code d'appairage contre un jeton | M1 |
| POST | `/conversations` | oui | ouvrir une conversation et obtenir son canal | M1 |
| POST | `/say` | oui | faire dire un texte à Jarvis, sans LLM | M1 |
| GET | `/audio/{asset}` | oui | télécharger un fichier audio | M1 |
| POST | `/turns` | oui | envoyer une prise audio | M2 |
| GET | `/conversations` | oui | lister les conversations | M4 |
| GET | `/conversations/{id}/messages` | oui | historique d'une conversation | M4 |

Le canal temps réel s'autorise par `POST /broadcasting/auth`, route standard de Laravel, avec le même jeton ([realtime](realtime.md)).

## GET /health

Toujours `200` si app répond ; l'état des dépendances est dans le corps. Ne révèle ni version de dépendance ni configuration.

```json
{
  "status": "ok",
  "app": { "ok": true, "commit": "a1b2c3d" },
  "db": { "ok": true },
  "ai": { "ok": true, "stt": true, "tts": true },
  "llm": { "driver": "deepseek", "configured": true, "ollama": true }
}
```

`status` vaut `degraded` dès qu'un élément est à `false`. Un appel dépendant ne dépasse pas 2 s : au-delà, l'élément est `false`.

## POST /devices/pair

Sans authentification. Limité à 5 essais par minute et par adresse.

```json
{ "code": "482913", "name": "iPhone d'Elias", "platform": "ios" }
```

- `code` : six chiffres, affiché par `php artisan dumont:pair` sur le Mac, valable 5 minutes, à usage unique.
- `platform` : `ios`, `android` ou `other`.

`201` :

```json
{ "token": "1|Xy…", "device": { "id": 1, "name": "iPhone d'Elias", "platform": "ios" } }
```

`422` si le code est faux, expiré ou déjà utilisé. `429` au-delà de la limite.

## POST /conversations

Corps vide. `201` :

```json
{ "id": "01J8Z3…", "channel": "conversation.01J8Z3…" }
```

La PWA garde la conversation ouverte tant qu'elle est au premier plan et en ouvre une nouvelle après 30 minutes d'inactivité.

## POST /say

Fait parler Jarvis sans passer par le LLM : sert au M1 et aux messages système. Le traitement est asynchrone.

```json
{ "conversation_id": "01J8Z3…", "text": "Bonjour, je suis Jarvis." }
```

- `text` : 1 à 2 000 caractères, découpé en phrases par l'orchestrateur.

`202` :

```json
{ "turn_id": "01J8Z4…" }
```

Événements émis ensuite : `tts.chunk` pour chaque phrase, puis `turn.done` ; `turn.failed` en cas d'échec.

## POST /turns

Envoie une prise audio entière, à la fin de la prise ([ADR-012](../adr/012-audio-http-evenements-ws.md)). Le traitement est asynchrone.

`multipart/form-data` :

| Champ | Type | Obligatoire | Note |
|---|---|---|---|
| `conversation_id` | ULID | oui | conversation ouverte de l'appareil |
| `audio` | fichier | oui | `audio/webm`, `audio/mp4` ou `audio/wav` ; 60 s et 5 Mo au plus |
| `mode` | texte | non | `chat` par défaut ; `echo` répète la transcription sans LLM |
| `released_at` | entier | non | horodatage du relâché, en millisecondes, pour les mesures |

`202` :

```json
{ "turn_id": "01J8Z5…" }
```

`422` si l'audio manque, dépasse les limites ou si la conversation est close. Événements émis ensuite : `transcript.final`, puis en mode `chat` `assistant.delta`, `tts.chunk`, `assistant.done`, et enfin `turn.done` ; `turn.failed` en cas d'échec.

## GET /audio/{asset}

Rend le fichier audio (`audio/wav` pour une synthèse, format d'origine pour une prise), à condition qu'il appartienne à l'utilisateur de l'appareil. `404` sinon. Les URL annoncées par `tts.chunk` pointent ici.

## GET /conversations

`200`, les 50 plus récentes :

```json
{ "data": [ { "id": "01J8Z3…", "started_at": "2026-09-21T14:52:00+02:00", "turns": 6, "preview": "Quelle heure est-il ?" } ] }
```

## GET /conversations/{id}/messages

`200`, dans l'ordre chronologique :

```json
{
  "data": [
    { "id": 41, "turn_id": "01J8Z5…", "role": "user", "content": "Quelle heure est-il ?", "audio": ["01J8Z6…"], "at": "2026-09-21T14:52:03+02:00" },
    { "id": 42, "turn_id": "01J8Z5…", "role": "assistant", "content": "Il est quatorze heures cinquante-deux.", "audio": ["01J8Z7…"], "provider": "deepseek", "model": "deepseek-flash", "at": "2026-09-21T14:52:05+02:00" }
  ]
}
```
