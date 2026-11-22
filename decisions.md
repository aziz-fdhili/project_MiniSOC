# Décisions d'architecture et de cadrage — MiniSOC

**Remarque de cadrage :** MiniSOC est une application légère de supervision défensive en laboratoire. Elle importe des journaux SSH et HTTP, normalise les événements, exécute trois règles de corrélation transparentes, affiche un tableau de bord et conserve l'historique et les alertes dans SQLite, avec export CSV. Une alerte constitue un indice documenté pour prioriser les vérifications, et non une conclusion qu'une attaque a réussi. Aucun agent intrusif, accès distant, blocage ou réponse automatique, aucune collecte en production et aucune promesse d'exhaustivité ne sont prévus. Les données utilisées sont synthétiques.

## 1. Choix techniques

* Type d'application : **application web locale**, ouverte dans le navigateur. Pas de bibliothèque d'interface à apprendre, et les tableaux, filtres et exports sont rapides à produire avec le temps imparti (18 à 20 h).
* Langage et outils : **Python 3, `sqlite3`, Streamlit**, tests avec `pytest`. Python et sqlite3 sont déjà maîtrisés ; seuls Streamlit et les expressions régulières sont à apprendre.
* Système de développement : Windows (VS Code, PowerShell).

## 2. Formats de logs et normalisation temporelle (UTC ISO 8601)

* Journaux SSH (syslog, `auth.log` Debian/Ubuntu) : exemple synthétique `Nov 16 10:23:45 srv01 sshd[1234]: Failed password for root from 203.0.113.10 port 52344 ssh2`. L'année et le fuseau horaire sont absents de la ligne.
* Journaux HTTP (format Combined Apache/Nginx, CLF accepté) : exemple synthétique `[16/Nov/2026:10:23:45 +0000]`. L'année et le décalage de fuseau (`+0000`) sont présents.
* Standard retenu : conversion de tous les horodatages en **UTC ISO 8601** (`AAAA-MM-JJTHH:MM:SSZ`) pour homogénéiser le tri, le filtrage et les fenêtres de corrélation.
* Hypothèses explicites : pour SSH, l'année est un paramètre d'import (2026 par défaut) et le fuseau est supposé UTC. Le fuseau affiché est indiqué dans l'interface. La ligne brute est conservée pour permettre la vérification sans falsifier l'origine du log.
* Champs manquants : IP absente enregistrée comme NULL ; referer et user-agent absents (CLF) enregistrés comme NULL.

## 3. Schéma conceptuel des données

* **Événement** : événement normalisé issu d'une ligne de log. Il contient l'horodatage UTC, la source (`ssh` ou `http`), le fichier, l'IP (nullable), l'utilisateur éventuel, le type, la gravité, le statut HTTP éventuel, un résumé lisible et la ligne brute.
* **Alerte** : indice documenté produit par une règle de corrélation (R1 répétitions SSH, R2 erreurs HTTP, R3 pic de volume). Elle contient la règle, la priorité, la période observée (`first_seen`, `last_seen`), la fenêtre, l'IP ou la source concernée, le volume, le seuil et les événements qui l'étayent. Elle n'a pas valeur de preuve de compromission.
* **Import** : trace d'un chargement de fichier. Elle contient le nom du fichier, la source, la date d'import, le nombre de lignes acceptées et rejetées, et le résultat.

## 4. Gravité, lignes invalides, réimports et export CSV

* Gravité d'un événement : `faible` (échecs et connexions SSH, HTTP 1xx à 4xx), `moyenne` (HTTP 5xx), `inconnue` lorsque l'information est absente ou illisible. Critères pédagogiques, ajustables après les tests.
* Priorité d'une alerte : `faible` (signal isolé), `moyenne` (seuil d'une règle atteint), `élevée` (plusieurs critères réunis ou dépassement marqué, à investiguer).
* Lignes invalides ou incomplètes : un échec de parsing n'interrompt pas l'import ; la ligne est rejetée, comptée avec son motif dans le bilan et tracée pour inspection.
* Réimports et doublons : un événement identique (même source, horodatage, IP et ligne brute) n'est pas inséré deux fois ; le bilan indique le nombre de doublons ignorés ; l'historique n'est jamais écrasé en silence.
* Export CSV : disponible pour les alertes et pour les événements filtrés. Encodage UTF-8 avec BOM (lecture correcte des accents dans Excel sous Windows), séparateur virgule, colonnes explicites, mêmes filtres et même période que l'écran. Protection contre l'injection de formules : préfixe d'une apostrophe pour les cellules commençant par `=`, `+`, `-`, `@`, une tabulation ou un retour chariot (recommandation OWASP). Limite connue : cette protection n'est pas universelle et peut perdre son effet dans Excel après réouverture et réenregistrement du fichier.

## 5. Sources et références consultées

* Documentation officielle Python (`datetime`, `sqlite3`, `csv`) — consultée le `21/11/2026` — https://docs.python.org/3/
* Documentation officielle Streamlit — consultée le `21/11/2026`— https://docs.streamlit.io/
* Documentation officielle SQLite — consultée le `AAAA-MM-JJ` — https://www.sqlite.org/docs.html
* Apache HTTP Server 2.4, module `mod_log_config` (formats CLF et Combined) — consultée le `21/11/2026` — https://httpd.apache.org/docs/2.4/mod/mod_log_config.html
* Page de manuel de `sshd` (OpenSSH, version Ubuntu) — consultée le `21/11/2026` — https://manpages.ubuntu.com/manpages/focal/en/man8/sshd.8.html
* OWASP, CSV Injection — consultée le `21/11/2026` — https://owasp.org/www-community/attacks/CSV_Injection
* Elastic Common Schema, référence des champs (inspiration des noms de champs) — consultée le `21/11/2026` — https://www.elastic.co/docs/reference/security/fields-and-object-schemas/siem-field-reference
