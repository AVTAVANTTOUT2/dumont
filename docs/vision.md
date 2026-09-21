# Vision

## Le produit

Dumont est un assistant vocal nommé Jarvis, porté au bras. L'équipier parle ; le brassard enregistre ; Laravel fait transcrire, interroge un modèle de langage (DeepSeek en ligne ou Ollama en local) et renvoie la réponse en voix, en texte et, quand la donnée a une forme, en visuel. Une seule application porte le chemin principal, tout le reste est un service appelé.

Pour le prototype, le brassard est le téléphone de l'utilisateur, avec une application web installée sur l'écran d'accueil ([ADR-009](adr/009-telephone-pwa-brassard.md)). Tout le calcul tourne sur un Mac Apple Silicon ([ADR-003](adr/003-trois-unites-un-mac.md)).

## Le prototype fonctionnel

Il est atteint quand, depuis le téléphone :

1. on parle à Jarvis, d'abord en appuyant sur un bouton, puis mains libres ;
2. il comprend et répond à voix haute, avec sa voix, en moins de trois secondes ;
3. il exécute une poignée de commandes (notes, rappels, état du système, bascule de modèle) et les affiche en cartes ;
4. chaque échange est journalisé, consultable et réécoutable ;
5. un scénario de trois minutes se joue sans toucher le Mac.

Le chemin pour y arriver est dans la [feuille de route](roadmap.md).

## Les trois usages visés par la V1

| Usage | Entrée | Sortie | Au prototype |
|---|---|---|---|
| Notes | voix continue | note structurée, horodatée, signée | notes simples par commande vocale (M5) |
| Communication | voix, messages | transcription, résumé, décisions, rapport | non |
| Restitution | question en langage naturel | réponse vocale, graphique, source citée | réponse vocale sans source (M3) |

## Ce que la plateforme fait

- Enregistre la voix sur le brassard et la fait transcrire en local.
- Route la demande vers un provider LLM interchangeable : DeepSeek par API, Ollama en local, même contrat.
- Exécute des outils déclarés en PHP : créer une note, programmer un rappel, donner l'état du système, changer de modèle.
- Rend la réponse en voix synthétisée phrase par phrase, en texte, et en cartes sur le tableau de bord du brassard.
- Journalise chaque interaction en ajout seul : prise audio, transcription, modèle, outils, réponse, synthèse.
- Expose un back-office généré avec Filament, pas écrit à la main.

## Hors périmètre

Du prototype :

- recherche documentaire (RAG), télémétrie, graphiques ;
- brassard physique dédié, plusieurs appareils, intercom ;
- notifications push, écoute écran verrouillé, mot d'activation acoustique ;
- tout hébergement hors du Mac.

De la V1, repris du cadrage :

- pas de commande d'organe physique : la plateforme propose, l'humain exécute ;
- pas de gestion de flotte, pas de multi-tenant ;
- pas d'entraînement ni de fine-tuning de modèle ;
- pas d'application mobile native : le brassard affiche une page web ;
- pas de haute disponibilité : un seul nœud, redémarrage automatique, sauvegarde nocturne.

## Principes directeurs

Ces règles tranchent par avance l'essentiel des débats d'implémentation. Elles priment sur la préférence personnelle.

1. **Un seul langage sur le chemin principal.** Tout ce qui touche la requête utilisateur est en PHP. Python n'existe que dans l'unité ai, parce que les modèles audio n'ont pas d'équivalent PHP.
2. **Le framework fait le travail.** Aucun code écrit là où Laravel, Reverb ou Filament couvre déjà le besoin. Un composant maison se justifie en revue.
3. **Les décisions sont écrites avant le code.** Les ADR ne se rediscutent pas pendant un jalon, seulement entre deux.
4. **Chaque domaine a un propriétaire.** Pas de fichier sans propriétaire ([structure](structure.md)).
5. **Tolérance sur la qualité, pas sur le contrat.** Une fonction de 80 lignes passe en revue ; une route renommée sans prévenir, non.
6. **Chaque jalon se montre sur le téléphone.** Ce qui ne se voit pas depuis le brassard ou Filament n'est pas fini.

## Les utilisateurs

- **Le porteur du brassard** — mains occupées, téléphone au poignet ou en main, interaction vocale d'abord. C'est lui qui dicte la conception.
- **L'ingénieur au poste fixe** — navigateur sur le Mac, consulte l'historique et le journal dans Filament.
- **L'administrateur** — appaire et révoque les téléphones, choisit les modèles actifs, relit le journal d'audit.

Au prototype, ces trois rôles sont tenus par la même personne.
