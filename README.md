# viper-onboarding-automation

GitHub Actions workflows triggered by Viper via [`repository_dispatch`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch).

## Event types

| `event_type`           | Workflow file                 | Purpose |
|------------------------|-------------------------------|---------|
| `onboarding-prepare`   | `onboarding-prepare.yml`      | Clone viper + feed-conversion, resolve latest FC tag, build proposed config, POST results to Viper. |
| `onboarding-apply`     | `onboarding-apply.yml`        | Fetch approved bundle from Viper, clone solution-jobs v3, branch, copy customer folder, open PR. |

Viper should send `client_payload` at minimum:

```json
{
  "draft_id": "<uuid>",
  "mode": "prepare"
}
```

(`mode` is redundant with `event_type` but useful for logging.) For `apply`, same `draft_id`.

## Repository secrets

Configure in **Settings → Secrets and variables → Actions** for this repo:

| Secret | Used by |
|--------|---------|
| `VIPER_BASE_URL` | Workflows calling back to Viper (e.g. `https://viper.example.com`). |
| `VIPER_ONBOARDING_HMAC_SECRET` | Shared secret to sign `POST` bodies to Viper ingest endpoints. |
| `GH_CLONE_TOKEN` | PAT with `repo` scope to clone **viper**, **feed-conversion**, and **solution-jobs** (read + push where needed). Optional if all repos are public and default `GITHUB_TOKEN` suffices (rare for private monorepos). |

## Manual test dispatch

Replace `ORG/REPO` with this repository’s full name:

```bash
gh api repos/ORG/REPO/dispatches \
  -f event_type=onboarding-prepare \
  -f client_payload='{"draft_id":"00000000-0000-0000-0000-000000000000"}'
```

## Related repositories

- **viper** — owns `OnboardingDraft`, review UI, and triggers these workflows.
- **feed-conversion** — read-only checkout at release tag during `prepare`.
- **solution-jobs v3** — destination repo for the onboarding PR during `apply`.
