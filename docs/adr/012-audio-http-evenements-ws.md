# ADR-012 — L'audio monte en HTTP, les événements descendent en WebSocket

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D2

## Contexte

Le cadrage envoyait l'audio du brassard en blocs de 280 ms par WebSocket (`audio.chunk`) et renvoyait le WAV synthétisé dans l'événement `tts.chunk`. Deux problèmes avec Reverb :

- Montée : un message émis par un client est un *whisper*, relayé aux autres abonnés du canal. Le traiter côté Laravel oblige à écouter dans le processus Reverb lui-même, qui ne doit jamais bloquer.
- Descente : Reverb plafonne un message à 10 Ko par défaut (`max_message_size`), comme le protocole Pusher, alors qu'une seconde de WAV à 24 kHz pèse environ 48 Ko.

## Décision

- **Montée :** une prise part entière, en un seul `POST /api/v1/turns` multipart, au relâché du bouton ([ADR-013](013-push-to-talk-dabord.md)).
- **Descente :** Reverb ne transporte que des événements JSON légers, de moins de 8 Ko. L'audio synthétisé est rangé sur le disque de l'app et annoncé par `tts.chunk` avec une URL ; le téléphone le télécharge avec son jeton et le joue.
- `audio.chunk` est retiré du contrat.

## Conséquences

- Pas de transcription partielle pendant qu'on parle : `transcript.partial` est réservé.
- Un aller-retour HTTP par phrase audio, négligeable sur le tailnet.
- Le traitement d'un tour vit dans le worker et non dans Reverb ; le WebSocket reste un simple haut-parleur.
- Si les mains libres exigent un jour l'audio en flux, un nouvel ADR le rétablira par un autre chemin que Reverb.
