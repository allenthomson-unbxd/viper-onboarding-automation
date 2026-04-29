# viper-onboarding-automation

GitHub Actions workflows triggered by Viper via [`repository_dispatch`](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#repository_dispatch).

## Event types

| `event_type`           | Workflow file                 | Purpose |
|------------------------|-------------------------------|---------|
| `onboarding-prepare`   | `onboarding-prepare.yml`      | Optionally read latest **feed-conversion** release tag, **GET** merged templates from **Viper** `prepare-context`, then **POST** `ingest-prepare`. |
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
| `VIPER_INTERNAL_BASE_URL` | Public base URL of Viper, no trailing slash (e.g. `https://search-pimapps.unbxd.io`). Used for `prepare-context` + `ingest-prepare`. GitHub cannot reach `http://localhost` without a tunnel. |
| `ONBOARDING_INGEST_HMAC_SECRET` | Same value as Viper env `ONBOARDING_INGEST_HMAC_SECRET`. Signs **GET** `prepare-context` (message = draft UUID) and **POST** `ingest-prepare` (message = raw JSON body). |
| `GH_CLONE_TOKEN` | PAT with `repo` (and `read:packages` if needed). Used for `gh api` on private `FEED_CONVERSION_REPO`, and later for clone/PR in **apply**. Optional if all APIs/repos are public. |
| `FEED_CONVERSION_REPO` | Optional, e.g. `unbxd/feed-conversion`. With `GH_CLONE_TOKEN`, workflow reads the **latest GitHub release tag** and passes it as `feed_conversion_tag` to `prepare-context`. |

If `VIPER_INTERNAL_BASE_URL` or `ONBOARDING_INGEST_HMAC_SECRET` is unset, **prepare** still runs but **skips** Viper calls (stub mode).

### Viper internal URLs (same HMAC secret as ingest)

| Method | Path | Signature |
|--------|------|-----------|
| GET | `/setup/app/internal/onboarding/<draft_id>/prepare-context/` | `X-Onboarding-Signature: sha256(HMAC(secret, utf8(draft_id)))` |
| POST | `/setup/app/internal/onboarding/<draft_id>/ingest-prepare/` | `X-Onboarding-Signature: sha256(HMAC(secret, raw_json_body))` |

`prepare-context` returns JSON including **`proposed_files`**: merged YAML/Argo text from **Viper’s** `cookiecutter_template/custom_transformer` (same layout as the feed-conversion cookiecutter). Optional query `feed_conversion_tag` is stored on the draft for traceability.

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
