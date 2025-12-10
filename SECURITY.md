# Security & Privacy Policy

## Scope
- Open science materials (JSON codes for cognitive tasks, docs, later anonymized/aggregated data/analyses).
- No raw data with personal identifiers belongs here.

## Data Hygiene
- Forbidden: names, emails, IDs, free text with identifiers, IPs, device data, raw logs.
- Allowed: anonymized/aggregated data with no re-identification risk.
- When in doubt: review before upload.

## Incident Handling (PII discovered?)
1. Remove/overwrite the file immediately (if in commit history: inform maintainers; do not force-push `main`).
2. Notify maintainers: <Contact>
3. Brief note in PR/Issue: what was affected, what was removed/replaced.

## Branch/PR Hygiene
- `main` is protected: PR required, 1 review, no force-push, no deletion.
- Use feature branches (`feat/...`, `docs/...`, `fix/...`) and fill the PR template.

## External Contributions / Replication
- Fork + branch in fork recommended; optional PR back.
- Replication via download/clone without PR is fine; please document versions/changes if contributing back.

## Tools / Checks (current)
- No mandatory CI status checks yet. CodeQL/tests can be added later.
- JSON codes: in `json-codes-cognitive-tasks/`; please describe/validate briefly.
