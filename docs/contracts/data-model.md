# Modèle de données du prototype

PostgreSQL 16 avec pgvector, base `dumont` ([ADR-006](../adr/006-postgres-pgvector-sans-redis.md)). Les migrations Laravel sont la seule source de vérité : pas de SQL écrit à la main hors migration, pas de modification directe en base. Propriétaire : D4.

## Tables de Laravel

Créées par le squelette et les paquets, non modifiées : `users` (étendue ci-dessous), `password_reset_tokens`, `sessions`, `cache`, `cache_locks`, `jobs`, `job_batches`, `failed_jobs`, `personal_access_tokens`.

La table `sessions` du cadrage entrait en collision avec celle de Laravel : l'échange s'appelle ici `conversations`, découpé en `turns`.

## Tables de Dumont

| Table | Contenu | Clés | Jalon |
|---|---|---|---|
| `users` | utilisateur, rôle, voix préférée | — | M1 |
| `devices` | téléphone appairé, porteur des jetons Sanctum | `user_id` | M1 |
| `conversations` | suite d'échanges sur un appareil | `user_id`, `device_id` | M1 |
| `turns` | un tour : une prise ou un texte à dire, et ses temps | `conversation_id` | M1 |
| `audio_assets` | fichier audio sur disque : prise ou phrase synthétisée | `turn_id` | M1 |
| `messages` | texte échangé, par rôle | `conversation_id`, `turn_id` | M3 |
| `events` | journal en ajout seul | `turn_id`, sujet polymorphe | M4 |
| `tool_calls` | appel d'outil, arguments, résultat, durée | `turn_id`, `message_id` | M5 |
| `notes` | note créée par la voix | `user_id`, `turn_id` | M5 |

### users

Colonnes ajoutées : `role` (`crew`, `engineer`, `admin`), `preferred_voice` (`jarvis-fr` par défaut). Le mot de passe ne sert qu'à Filament, sur le Mac.

### devices

`id`, `user_id`, `name`, `platform` (`ios`, `android`, `other`), `last_seen_at`, `revoked_at`, horodatages. Le modèle `Device` porte `HasApiTokens` : un jeton appartient à un appareil, et l'utilisateur authentifié d'une requête d'API est l'appareil. Révoquer un appareil supprime ses jetons.

Le code d'appairage ne vit pas en table : il est rangé dans le cache (en base), haché, pour 5 minutes.

### conversations

`id` (ULID), `user_id`, `device_id`, `status` (`open`, `closed`, `abandoned`), `closed_at`, horodatages.

### turns

`id` (ULID), `conversation_id`, `mode` (`say`, `echo`, `chat`), `status` (`queued`, `transcribing`, `thinking`, `speaking`, `done`, `failed`), `failed_stage`, `provider`, `model`, `fallback` (booléen), `stt_ms`, `llm_first_ms`, `tts_first_ms`, `total_ms`, `released_at`, horodatages.

Les colonnes de temps alimentent les [mesures](../mesures.md) : elles se remplissent dès le M1.

### audio_assets

`id` (ULID), `turn_id`, `kind` (`input` pour une prise, `speech` pour une synthèse), `seq` (ordre de la phrase, `null` pour une prise), `disk`, `path`, `mime`, `duration_ms`, `bytes`, `created_at`.

Les fichiers vivent sur le disque local de l'app, sous `storage/app/audio/{conversation}/{turn}/`, jamais en base. Au prototype, rien n'est purgé.

### messages

`id`, `conversation_id`, `turn_id`, `role` (`user`, `assistant`, `tool`), `content`, `provider`, `model`, `tokens_in`, `tokens_out`, `created_at`. Le texte transcrit d'une prise est un message `user` ; la réponse complète, un message `assistant`.

### events

`id`, `turn_id` (facultatif), `subject_type`, `subject_id`, `type`, `payload` (jsonb), `created_at`. Pas de `updated_at`.

Types au prototype : `turn.received`, `stt.done`, `llm.started`, `llm.fallback`, `llm.done`, `tts.done`, `tool.called`, `turn.done`, `turn.failed`, `device.paired`, `device.revoked`.

Un déclencheur PostgreSQL refuse `UPDATE`, `DELETE` et `TRUNCATE` sur la table ([ADR-015](../adr/015-journal-ajout-seul.md)). Toute correction est un nouvel événement.

### tool_calls

`id`, `turn_id`, `message_id`, `tool`, `arguments` (jsonb), `result` (jsonb), `ok`, `duration_ms`, `created_at`.

### notes

`id`, `user_id`, `turn_id`, `kind` (`observation`, `action`, `anomaly`), `body`, horodatages.

## Après le prototype

Viendront avec la V1 : `attachments`, `telemetry_points`, `documents` et `document_chunks` avec une colonne `vector(1024)` pour les embeddings bge-m3 ([ADR-008](../adr/008-embeddings-locaux.md)).
