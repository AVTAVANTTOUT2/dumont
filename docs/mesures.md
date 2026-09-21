# Mesures

Les chiffres qui décident. Chaque jalon qui touche la voix, le LLM ou la mémoire remplit sa colonne ; une mesure se prend sur le Mac de démonstration, avec le téléphone principal, en médiane de dix essais au moins (vingt pour le bout en bout). Les hypothèses de départ sont dans le [budget de latence](architecture.md#budget-de-latence).

## Latences

| Mesure | Méthode | Cible | M1 | M2 | M3 | M6 |
|---|---|---|---|---|---|---|
| TTS, phrase de quinze mots | `POST /tts` direct sur la façade | < 1,0 s | | | | |
| STT, cinq secondes d'audio | `POST /stt` direct sur la façade | < 0,8 s | | | | |
| Premier fragment, deepseek-flash | horodatage dans le worker | < 1,0 s | | | | |
| Premier fragment, Ollama local | horodatage dans le worker | < 1,5 s | | | | |
| Fin de prise → premier son, DeepSeek | horodatages du téléphone, 20 questions de référence | < 3,0 s | | | | |
| Fin de prise → premier son, Ollama | idem | < 4,0 s | | | | |
| Fin de prise → texte transcrit affiché | idem | < 1,5 s | | | | |

## Mémoire

| Mesure | Méthode | Cible | M1 | M2 | M3 | M6 |
|---|---|---|---|---|---|---|
| Mémoire utilisée en conversation | Moniteur d'activité, pic sur 20 questions | < 20 Go | | | | |
| Mémoire de la façade voix | `ps` sur les deux processus Python | < 6 Go | | | | |
| Swap pendant une conversation | `sysctl vm.swapusage` avant et après | 0 | | | | |

## Environnement de mesure

| Élément | Valeur |
|---|---|
| Mac | Apple M5, 24 Go |
| macOS | à relever |
| Téléphone | à relever ([#31](https://github.com/AVTAVANTTOUT2/dumont/issues/31)) |
| Réseau | Tailscale, même Wi-Fi puis 4G |
| Commit de jarvis-voice | à relever |
| Modèle Ollama | à relever ([#32](https://github.com/AVTAVANTTOUT2/dumont/issues/32)) |
