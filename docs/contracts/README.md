# Contrats

Ce qui relie les unités et les domaines. Un contrat change dans la même PR que le code qui le change ; renommer ou changer le sens d'un élément existant exige l'étiquette `contract` et deux relecteurs ([CONTRIBUTING](../../CONTRIBUTING.md#étiquettes)).

| Contrat | Entre | Propriétaire |
|---|---|---|
| [API REST](api.md) | téléphone → app | D2, D4 pour l'appairage |
| [Événements temps réel](realtime.md) | app → téléphone | D2 |
| [Service ai](ai-service.md) | app → unité ai | D3 |
| [Modèle de données](data-model.md) | app → db | D4 |

Colonne « Jalon » : le jalon où l'élément apparaît. Tant qu'il n'est pas livré, il peut encore bouger sans étiquette `contract`, à condition de mettre à jour cette page.
