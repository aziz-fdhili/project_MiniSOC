# MiniSOC

Outil local de supervision défensive pour laboratoire : il lit des journaux SSH et HTTP, les convertit en événements d'un format commun, applique des règles de corrélation et présente les résultats dans un tableau de bord, avec historique SQLite et export CSV.

## Cadre d'utilisation

MiniSOC est un projet pédagogique destiné à un usage local et défensif. Une alerte est un indice documenté qui aide à choisir quoi vérifier en premier ; elle ne prouve pas qu'une attaque a réussi. L'outil n'installe aucun agent, n'accède à aucune machine distante, ne bloque rien, ne réagit pas automatiquement et ne collecte aucune donnée en production. Les journaux utilisés sont synthétiques.

## Semaine 1 : cadrage

Contenu actuel du dépôt :

* `docs/decisions.md` : choix d'architecture, formats de logs, normalisation des dates, schéma de données, gravité, export CSV et sources.

Choix retenus :

* Application web locale, en Python avec Streamlit.
* Base de données SQLite.
* Logs SSH au format `auth.log` Debian/Ubuntu et logs HTTP au format Apache/Nginx Combined.
* Dates stockées en UTC, au format ISO 8601.
* Export CSV en UTF-8.

## Suite du projet

Ingestion et SQLite (semaine 2), règles et interface (semaine 3), scénarios et présentation (semaine 4). Ce README sera complété à chaque étape.
