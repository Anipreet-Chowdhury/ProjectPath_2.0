## Branching model

    - **main** = always deployable / always “green”
    - Work happens in feature branches (examples below):
        *feat/week1-foundations
        *feat/pdf-extraction
        *fix/celery-retry-bug
    - Merge via PR
---

## Commit discipline

### Commit size
    
    * Prefer 5–20 files per commit, not 80.
    * If a commit touches unrelated areas (docs + code + config), split it.

### Commit message style

    * chore: tooling/config/CI
    * docs: documentation only
    * feat: new behavior
    * fix: bug fix
    * refactor: code movement / cleanup (no behavior change)
    * test: tests only

### Examples:

    - docs: add v2 architecture and rollout plan
    - chore: initialize monorepo folder structure
    - feat(api): add job state machine enum
    - feat(web): add job status polling UI