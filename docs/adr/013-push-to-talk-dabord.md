# ADR-013 — Push-to-talk d'abord, mains libres ensuite

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D1

## Contexte

Le premier objectif est que Jarvis parle et nous entende, de façon fiable. La détection de parole et le mot d'activation ajoutent des faux départs, des coupures et des réglages à chaque pièce.

## Décision

- Jusqu'au M5, une prise égale un appui sur le bouton PARLER : appui maintenu, ou bascule pour les longues prises. 60 secondes au plus.
- Au M6, un mode mains libres s'ajoute : détection de parole dans le navigateur (Silero en ONNX), et demande retenue seulement si « Jarvis » est prononcé, vérifié après la transcription, comme dans jarvis-office.
- Pas de mot d'activation acoustique au prototype.
- Semi-duplex : Jarvis n'écoute pas pendant qu'il parle. Un appui sur PARLER coupe sa voix.

## Conséquences

- Interaction à une main jusqu'au M6.
- Pas de *barge-in* : on ne coupe pas Jarvis en parlant par-dessus.
- La même route `POST /api/v1/turns` sert aux deux modes : en mains libres, c'est la détection de parole qui découpe les prises.
