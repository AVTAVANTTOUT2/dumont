# ADR-001 — Laravel 13 sur FrankenPHP comme socle unique

- **Statut :** Acceptée
- **Date :** 2026-09-21
- **Domaine :** D5

## Contexte

Il faut du web, du temps réel, une file de jobs, un ORM, une authentification par jeton et un back-office, avec peu de temps. Le cadrage visait Laravel 12. Au 21 septembre 2026, la version courante est Laravel 13 (13.32) ; Laravel 12 n'a plus que des correctifs de sécurité, jusqu'en février 2027.

## Décision

Laravel 13 sur FrankenPHP, PHP 8.5, dans le conteneur `app`. Pas de Symfony, pas de micro-framework, pas de découpage en microservices PHP. FrankenPHP tourne en mode classique au prototype. Paquets de première partie : Sanctum, Reverb, Pest.

## Conséquences

- Le framework impose ses conventions, donc peu de décisions à prendre en route. On accepte sa magie et on ne se bat pas contre elle.
- PHP 8.5 est aussi la version installée sur le Mac : les outils en ligne de commande et le conteneur parlent la même langue.
- Le mode worker de FrankenPHP (Octane) reste une optimisation possible plus tard, pas un prérequis.

## Alternatives écartées

- Laravel 12 : déjà hors de sa période de corrections, pour un projet qui démarre.
- PHP-FPM derrière Nginx : un processus et une configuration de plus, et un flux de réponse moins simple que FrankenPHP.
