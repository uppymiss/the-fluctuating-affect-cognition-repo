# A Prospective Study on Cognitive and Emotional Fluctuations Over the Course of a Month

**Repository DOI (Zenodo):**
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.17871655.svg)](https://doi.org/10.5281/zenodo.17871655)


**OSF Project DOI:**
[![OSF icon](https://raw.githubusercontent.com/uppymiss/the-fluctuating-affect-cognition-repo/main/media/osf_icon_105x40.svg)](https://doi.org/10.17605/OSF.IO/XRF7U)

*OSF logo (c) Center for Open Science. License: CC BY 4.0.*

--------

## Purpose

Open science companion repository for the study. Contains:
- lab.js source files (JSON) for cognitive tasks (Stroop and Flanker) used in SoSci Survey
- Documentation on how to embed lab.js tasks in SoSci Survey and store trial-level data
- Community files (README, CONTRIBUTING, SECURITY)
- Later: analysis scripts and anonymized data

## Cognitive Tasks

Two tasks are administered across 9 measurement points (MZP1_1 through MZP3_3):

### Stroop Task
- Color-word interference paradigm (German: rot, gruen, blau, orange)
- Congruent and incongruent trials
- Timing: 500 ms fixation, 900 ms stimulus timeout (test), 650 ms inter-trial interval
- Practice: unlimited response time with feedback

### Flanker Task
- Arrow flanker paradigm (left/right arrows, congruent/incongruent)
- Timing: 200 ms fixation, 850 ms stimulus timeout (test)
- Practice: unlimited response time with feedback

### Task Variants

| Variant | Practice Blocks | Test Blocks | Used For |
|---------|----------------|-------------|----------|
| 2t (two-trial) | 2 x 12 trials | 3 x 20 trials | MZP1_1 only (first session) |
| 1t (one-trial) | 1 x 12 trials | 3 x 20 trials | MZP1_2 through MZP3_3 |

The first measurement point includes an extra practice block so participants can familiarize themselves with the task.

### Data Transfer

Trial-level data (block, trial number, congruency, accuracy, RT, stimulus, response) is transferred from the lab.js iframe to SoSci Survey via `window.parent.postMessage()`. See [docs/HOWTO_labjs_SoSci_Integration.md](docs/HOWTO_labjs_SoSci_Integration.md) for the full integration guide.

## Repository Structure

```
.
├── json-codes-cognitive-tasks/
│   ├── Flanker/              -- 9 Flanker task JSON files (lab.js format)
│   │   ├── k104_flanker_2t_lang_mzp1_1-*.study.json   (2t variant)
│   │   ├── k204_flanker_1t_lang_mzp1_2-*.study.json
│   │   ├── ...
│   │   └── k904_flanker_1t_lang_mzp3_3-*.study.json
│   │
│   └── Stroop/               -- 9 Stroop task JSON files (lab.js format)
│       ├── k102_stroop_2t_lang_mzp1_1-*.study.json    (2t variant)
│       ├── k202_stroop_1t_lang_mzp1_2-*.study.json
│       ├── ...
│       └── k902_stroop_1t_lang_mzp3_3-*.study.json
│
├── docs/
│   └── HOWTO_labjs_SoSci_Integration.md  -- Full guide: embedding lab.js in SoSci
│
├── media/                    -- Icons and images for documentation
├── .github/                  -- PR template, workflows, issue templates
├── CONTRIBUTING.md
├── SECURITY.md
├── CODE_OF_CONDUCT.md
└── LICENSE
```

### File Naming Convention

`k[MZP-Nr][Task-Nr]_[task]_[variant]_lang_[mzp]-[date].study.json`

- **k[MZP-Nr]**: Section number (1-9, mapping to MZP1_1 through MZP3_3)
- **[Task-Nr]**: 02 = Stroop, 04 = Flanker
- **[task]**: `stroop` or `flanker`
- **[variant]**: `1t` (one practice block) or `2t` (two practice blocks)
- **lang**: long version (3 test blocks)
- **[mzp]**: measurement point identifier

## Branching & Reviews
- Protected branch: **main** (PR required, 1 review, no force-push, no deletion)
- Working branches: short-lived, e.g., `feat/...`, `docs/...`, `fix/...`
- Merge flow: branch -> PR (use template) -> review -> merge into `main`

## Contribution Model
- **Internal team:** create a branch in this repo, PR into `main`.
- **External (replication/improvements):** fork + branch in fork; optional PR back. Pure replication may also clone/download without PR.
- No PII/identifiers/free text with personal data in the repo.

## Data & Privacy
- Only anonymized/aggregated data may be uploaded.
- Raw data with personal identifiers stays out of the repo.
- If you find identifiers: remove/overwrite immediately, notify maintainers.

## Links
- GitHub: https://github.com/uppymiss/the-fluctuating-affect-cognition-repo
- OSF project/preregistration: [OSF link](https://osf.io/xrf7u/overview)
- CONTRIBUTING: ./CONTRIBUTING.md
- Security/Privacy: ./SECURITY.md
- SoSci Integration Guide: [docs/HOWTO_labjs_SoSci_Integration.md](docs/HOWTO_labjs_SoSci_Integration.md)
