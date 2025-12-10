# Contributing

## Branches
- Work on feature branches (`feat/...`, `fix/...`, `docs/...`), then PR into `main`.
- `main` is protected (PR required, 1 review, no force-push, no deletion).

## PRs
- Fill out the PR template (Summary, Testing, Notes).
- At least 1 review before merge.
- If “Require linear history” is enabled: prefer rebase & merge (no merge commits).

## Data Hygiene
- Do not commit PII/raw data/IDs/free text with identifiers.
- Anonymized/aggregated data only, with clear documentation.
- If a violation occurs: remove/overwrite the file(s) immediately and inform maintainers.

## JSON Codes (SoSci Survey)
- Location: `json-codes-cognitive-tasks/`
- Briefly describe what each JSON does; if relevant, include validation/notes.

## External Contributions / Replication
- Fork + branch in the fork; optional PR back.
- Replication can also be done via clone/download without PR; please document versions/changes if contributing back.

## Communication
- Use Issues for proposals/bugs/todos.
- Keep PR titles/descriptions concise and specific.

## Structure
- `json-codes-cognitive-tasks/`- JSON codes for cognitive tasks (SoSci integration)
- `.github/` – PR template
- `README.md`, `CONTRIBUTING.md`, `SECURITY.md`
- (Later) `analysis/` for additional scripts
- (Later) `data/` – only anonymized/aggregated data (no raw data or identifiers)
