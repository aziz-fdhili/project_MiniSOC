# MiniSOC

A local defensive monitoring tool for laboratory use: it reads SSH and HTTP logs, converts them into events in a common format, applies correlation rules and displays the results in a dashboard, with a SQLite history and CSV export.

## Scope of use

MiniSOC is an educational project intended for local, defensive use. An alert is a documented indication that helps decide what to check first; it does not prove that an attack succeeded. The tool installs no agent, accesses no remote machine, blocks nothing, does not respond automatically and does not collect any production data. The logs used are synthetic.

## Week 1: scoping

Current contents of the repository:

* `docs/decisions.md`: architecture choices, log formats, date normalization, data schema, severity, CSV export and sources.

Choices made:

* Local web application, in Python with Streamlit.
* SQLite database.
* SSH logs in the Debian/Ubuntu `auth.log` format and HTTP logs in the Apache/Nginx Combined format.
* Dates stored in UTC, in ISO 8601 format.
* CSV export in UTF-8.

## Next steps

Ingestion and SQLite (week 2), rules and interface (week 3), scenarios and presentation (week 4). This README will be updated at each stage.