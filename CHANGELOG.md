# Journal des versions

Toutes les évolutions notables de Dumont sont consignées ici. Format inspiré de [Keep a Changelog](https://keepachangelog.com/fr/1.1.0/), numérotation [SemVer](https://semver.org/lang/fr/).

Chaque jalon terminé produit une version : M0 → 0.1.0, M1 → 0.2.0, et ainsi de suite jusqu'à M6 → 0.7.0, le prototype fonctionnel. La 1.0.0 viendra avec la V1.

Une PR qui change quelque chose de visible ajoute sa ligne sous « Non publié », dans la bonne rubrique : **Ajouté**, **Modifié**, **Corrigé**, **Retiré**, **Sécurité**. Une ligne par PR, écrite pour quelqu'un qui utilise Jarvis, pas pour quelqu'un qui lit le diff.

## [Non publié]

## [0.0.0] — 2026-09-21

### Ajouté

- Documentation initiale : vision, architecture en trois unités (`ai` en natif, `app` et `db` dans Docker), structure du dépôt, feuille de route M0 à M6.
- Dix-neuf ADR, qui remplacent les quinze du document de cadrage du 21 septembre 2026.
- Contrats : API REST `/api/v1`, événements temps réel, service `ai`, modèle de données du prototype.
- Spécification du brassard sur téléphone (PWA) et grille de mesures.
- Méthode de développement, modèle de PR et modèles d'issue.
