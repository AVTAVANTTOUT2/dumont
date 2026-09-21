# Structure du dépôt

Un monorepo, trois unités, un propriétaire par répertoire. Cette page décrit l'arborescence **cible** : aujourd'hui seuls la documentation et `.github/` existent, le reste arrive au jalon indiqué.

## Arborescence

```text
dumont/
├── README.md                       présentation, démarrage
├── CHANGELOG.md                    journal des versions
├── CONTRIBUTING.md                 méthode de développement
├── CLAUDE.md                       consignes pour les agents de code
├── Makefile                        commandes du quotidien                     M0 · D5
├── compose.yaml                    conteneurs app et db                       M0 · D5
├── .env.example                    variables, sans secret                     M0 · D5
├── .github/
│   ├── pull_request_template.md    format des PR
│   ├── ISSUE_TEMPLATE/             tâche, bug, décision
│   └── workflows/ci.yml            intégration continue                       M0 · D5
├── docs/
│   ├── vision.md · architecture.md · structure.md · roadmap.md
│   ├── brassard-pwa.md · mesures.md
│   ├── adr/                        NNN-titre.md, une décision par fichier
│   └── contracts/                  api · realtime · ai-service · data-model
├── docker/                                                                    M0 · D5
│   ├── app/                        Dockerfile, Caddyfile, supervisord.conf
│   └── db/initdb/                  création de l'extension vector
├── scripts/                        doctor.sh, relevé des mesures              M0 · D5
├── ai/                             unité ai, en natif                         M0 · D3
│   ├── jarvis-voice/               sous-module privé, jamais modifié
│   ├── pyproject.toml              façade, Python 3.12, avec le STT
│   ├── tts/pyproject.toml          worker TTS, Python 3.14
│   ├── src/dumont_ai/
│   │   ├── server.py               FastAPI : /health, /stt, /tts
│   │   ├── stt.py                  ffmpeg vers WAV 16 kHz, puis Transcriber
│   │   ├── tts_client.py           pilote le worker TTS
│   │   └── tts_worker.py           tourne dans .venv-tts, charge Speaker une fois
│   └── tests/                      tests de contrat, moteurs simulés
└── app/                            Laravel 13                                 M0 · D5
    ├── app/
    │   ├── Http/Controllers/Api/V1/    routes /api/v1                         D2, D4
    │   ├── Events/                     événements temps réel                  D2
    │   ├── Jobs/                       ProcessTurn, SpeakText                 D2
    │   ├── Llm/                        LlmProvider, DeepSeek, Ollama          D2
    │   ├── Orchestrator/               tour de parole, découpage en phrases   D2
    │   ├── Tools/                      un outil par classe                    D2
    │   ├── Voice/                      client HTTP de la façade ai            D3
    │   ├── Models/                     Eloquent                               D4
    │   └── Filament/                   back-office                            D4
    ├── database/migrations/                                                   D4
    ├── resources/
    │   ├── views/band/                 la PWA du brassard                     D1
    │   ├── js/band/                    capture, lecture, temps réel           D1
    │   └── prompts/                    prompt système de Jarvis               D2
    ├── public/                         manifest.webmanifest, sw.js, icônes    D1
    ├── routes/                         api.php, channels.php                  D2
    └── tests/                          Pest : Feature, Unit
```

`app/app/` n'est pas une faute de frappe : le premier `app/` est le répertoire de l'unité, le second celui de Laravel.

## Propriétaires

| Chemin | Domaine | Jalon d'arrivée |
|---|---|---|
| `docs/` | celui du sujet traité ; D5 pour l'index | déjà là |
| `docs/contracts/` | D2 pour api et realtime, D3 pour ai-service, D4 pour data-model | déjà là |
| `.github/`, `docker/`, `scripts/`, `Makefile`, `compose.yaml` | D5 | M0 |
| `ai/` sauf `ai/jarvis-voice/` | D3 | M0 |
| `ai/jarvis-voice/` | personne : dépôt externe, mis à jour par montée de commit du sous-module | M0 |
| `app/resources/views/band`, `app/resources/js/band`, `app/public` | D1 | M1 |
| `app/app/Llm`, `Orchestrator`, `Tools`, `Jobs`, `Events`, `app/routes`, `app/resources/prompts` | D2 | M1 à M5 |
| `app/app/Voice` | D3 | M1 |
| `app/database`, `app/app/Models`, `app/app/Filament` | D4 | M1 à M4 |
| tout le reste de `app/` (config, bootstrap, composer) | D5 | M0 |

## Règles

- Rien à la racine hors outillage et documents de premier niveau.
- Un répertoire, un propriétaire. Un nouveau répertoire entre dans la table ci-dessus dans la PR qui le crée.
- Ne sont jamais versionnés : `.env`, `vendor/`, `node_modules/`, `public/build/`, `.venv-*`, les poids de modèles, les enregistrements audio.
- Le code de jarvis-voice ne se modifie pas depuis ce dépôt ([ADR-005](adr/005-jarvis-voice-sous-module.md)).
