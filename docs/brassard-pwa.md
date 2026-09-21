# Brassard : la PWA du téléphone

Le brassard (*gauntlet*) du prototype est le téléphone de l'utilisateur. Il sert de tableau de bord pour parler à Jarvis, l'entendre et le commander. C'est une application web installée sur l'écran d'accueil, servie par le conteneur app ([ADR-009](adr/009-telephone-pwa-brassard.md)). Domaine D1.

## Ce qu'elle fait

- Enregistre une prise au micro et l'envoie à Laravel.
- Affiche ce que Jarvis a compris, puis sa réponse qui s'écrit.
- Joue la voix de Jarvis, phrase par phrase, dans l'ordre.
- Montre l'état des services et, à partir du M5, les cartes que Jarvis pousse (note, rappel, état).

Elle ne parle qu'à Laravel, sur une seule origine HTTPS : l'API, la page et le WebSocket passent par `https://<mac>.<tailnet>.ts.net` ([ADR-010](adr/010-https-tailscale-serve.md)).

## Écrans

| Écran | Jalon | Contenu |
|---|---|---|
| Installation et appairage | M1 | comment l'installer sur iPhone ou Android ; saisie du code à six chiffres affiché sur le Mac |
| Console | M0 puis M1 à M5 | état des services, échange en cours, cartes, bouton PARLER |
| Historique | M4 | conversations passées, réécoute des réponses |
| Réglages | M5 | voix activée ou non, provider LLM, mode mains libres (M6), révocation de l'appareil |

## La console

```text
┌────────────────────────────────┐
│ JARVIS          ● ai ● llm ● db │  pastilles : GET /api/v1/health
├────────────────────────────────┤
│                                │
│  Vous                          │
│  « Quelle heure est-il ? »     │  transcript.final
│                                │
│  Jarvis                        │
│  Il est quatorze heures        │  assistant.delta puis assistant.done
│  cinquante-deux.               │
│  ▂▃▅▇▅▃  phrase 1 / 2          │  file des tts.chunk
│                                │
│  ┌──────────────────────────┐  │
│  │ Note ajoutée             │  │  ui.render (M5)
│  │ La porte du garage grince│  │
│  └──────────────────────────┘  │
│                                │
│          ╭──────────╮          │
│          │  PARLER  │          │  appui maintenu = une prise
│          ╰──────────╯          │
│  Historique          Réglages  │
└────────────────────────────────┘
```

Le bouton PARLER porte tout l'état de l'échange :

| État | Aspect | Déclencheur | Action d'un appui |
|---|---|---|---|
| Repos | bouton plein | `turn.done`, démarrage | commence une prise |
| Écoute | rouge, durée qui défile | appui | le relâché envoie la prise |
| Envoi | anneau qui tourne | relâché | aucune |
| Jarvis réfléchit | pulsation lente | `202`, `transcript.final` | annule l'affichage du tour |
| Jarvis parle | onde animée | premier `tts.chunk` | coupe la voix et revient au repos |
| Erreur | orange, message court | `turn.failed`, perte réseau | revient au repos |

Un mode bascule (un appui pour commencer, un appui pour finir) existe pour les longues prises. Une prise est coupée à 60 secondes, la limite de jarvis-voice.

## Lecture de la voix

1. Au premier appui sur PARLER, la PWA crée et reprend son `AudioContext` : iOS n'autorise le son qu'après un geste de l'utilisateur.
2. À chaque `tts.chunk`, elle télécharge le WAV (`fetch` avec le jeton), le décode avec `decodeAudioData` et le range dans une file selon `seq`.
3. Elle joue la phrase `seq = n` seulement après `n − 1`, en planifiant chaque source audio à la fin de la précédente : pas de blanc entre deux phrases.
4. Un appui pendant la lecture coupe toutes les sources et vide la file.

## Différences entre iPhone et Android

| Sujet | iPhone (Safari, PWA installée) | Android (Chrome) | Ce qu'on fait |
|---|---|---|---|
| Installation | Partager → « Sur l'écran d'accueil » | invite d'installation ou menu | écran d'aide selon la plateforme |
| Format du micro | `audio/mp4` (AAC) | `audio/webm;codecs=opus` | le client choisit avec `MediaRecorder.isTypeSupported` ; la façade accepte les deux par ffmpeg |
| Lecture audio | bloquée avant un geste | plus tolérant | `AudioContext` repris au premier appui |
| Permission du micro | peut être redemandée après relance de la PWA | mémorisée | message clair et bouton de relance si refusée |
| Vibration | absente | `navigator.vibrate` | l'événement `haptic` est ignoré sur iPhone |
| Écran allumé | Wake Lock API | Wake Lock API | verrou demandé pendant une conversation |
| Arrière-plan | micro coupé écran verrouillé | micro coupé écran verrouillé | pas d'écoute en arrière-plan au prototype |
| Notifications push | possibles pour une PWA installée | possibles | hors prototype |

Les deux plateformes sont visées ; on teste d'abord sur le téléphone principal ([#31](https://github.com/AVTAVANTTOUT2/dumont/issues/31)), l'autre avant chaque fin de jalon.

## Choix techniques

- Une page Blade plein écran (`resources/views/band`) et du JavaScript en modules ES natifs (`resources/js/band`), assemblés par Vite. Pas de framework front.
- Temps réel : Laravel Echo et `pusher-js`, autorisation du canal privé par le jeton d'appareil.
- Jeton d'appareil conservé dans `localStorage`, envoyé en `Authorization: Bearer` ([ADR-011](adr/011-jeton-appareil.md)).
- Service worker : met en cache la coquille (HTML, JS, CSS, icônes) pour un démarrage rapide. Aucune réponse d'API ni aucun audio n'est servi depuis le cache.
- Manifest : `display: standalone`, orientation portrait, thème sombre, nom « Jarvis ».
- Budget : moins de 60 Ko de JavaScript compressé, premier affichage en moins d'une seconde sur le réseau du tailnet.

## Hors prototype

Écoute écran verrouillé, notifications push, mode hors ligne, plusieurs appareils, molette et moteurs vibrants du brassard physique.
