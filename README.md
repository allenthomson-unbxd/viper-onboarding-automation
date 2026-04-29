# viper-onboarding-automation

GitHub Actions workflows triggered by Viper via [`repository_dispatch`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch).

## Event types

| `event_type`           | Workflow file                 | Purpose |
|------------------------|-------------------------------|---------|
| `onboarding-prepare`   | `onboarding-prepare.yml`      | Clone viper + feed-conversion, resolve latest FC tag, build proposed config, POST results to Viper. |
| `onboarding-apply`     | `onboarding-apply.yml`        | Fetch approved bundle from Viper, clone solution-jobs v3, branch, copy customer folder, open PR. |

Viper sends `client_payload` on successful site-config save (when `GITHUB_ONBOARDING_REPO` and `GITHUB_ONBOARDING_TOKEN` are set), for example:

```json
{
  "draft_id": "<uuid>",
  "site_key": "<unbxd site key>"
}
```

For `onboarding-apply`, use the same `draft_id` after the operator approves the draft in Viper.

## Repository secrets

Configure in **Settings → Secrets and variables → Actions** for this repo:

| Secret | Used by |
|--------|---------|
| `VIPER_INTERNAL_BASE_URL` | Public base URL of Viper, no trailing slash (e.g. `https://search-pimapps.unbxd.io`). Used for `POST …/setup/app/internal/onboarding/<draft_id>/ingest-prepare/`. GitHub cannot reach `http://localhost`; use a deployed URL or a tunnel. |
| `ONBOARDING_INGEST_HMAC_SECRET` | Same value as Viper env `ONBOARDING_INGEST_HMAC_SECRET`. Signs the raw JSON body for ingest. |
| `GH_CLONE_TOKEN` | PAT with `repo` scope to clone **viper**, **feed-conversion**, and **solution-jobs** (read + push where needed). Optional until clone/PR steps are implemented; optional if repos are public and `GITHUB_TOKEN` is enough. |

If `VIPER_INTERNAL_BASE_URL` or `ONBOARDING_INGEST_HMAC_SECRET` is unset, **prepare** still runs but **skips** the ingest step (stub mode).

## Manual test dispatch

Replace `ORG/REPO` with this repository’s full name:

```bash
gh api repos/ORG/REPO/dispatches \
  -f event_type=onboarding-prepare \
  -f client_payload='{"draft_id":"00000000-0000-0000-0000-000000000000","site_key":"your-site-key"}'
```

## Related repositories

- **viper** — owns `OnboardingDraft`, review UI, and triggers these workflows.
- **feed-conversion** — read-only checkout at release tag during `prepare`.
- **solution-jobs v3** — destination repo for the onboarding PR during `apply`.
