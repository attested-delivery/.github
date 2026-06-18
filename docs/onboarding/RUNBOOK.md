# attested-delivery — org constitution runbook

Materializes the org `attested-delivery` as a secure, attested-workflow
reference org. All content is sanitized of non-`attested-delivery` names.

## What is built (in this working dir, pushed where noted)

| Path | Purpose | Pushed to |
| --- | --- | --- |
| `repos/.github/` | Org community-health defaults + reusable attested quality-gate workflows | `attested-delivery/.github` (public, template) |
| `repos/attested-pipeline-template/` | Language-agnostic attested release pipeline template | `attested-delivery/attested-pipeline-template` (public, template) |
| `repos/rust-template/` | Sanitized Rust attested template | `attested-delivery/rust-template` (public, template) |
| `docs-site/` | Astro Starlight docs site (overview, 10 concepts, 5 specs, 8 ADRs) | `attested-delivery/docs` (public) — Pages: https://attested-delivery.github.io/docs/ |
| `app/manifest.json`, `app/create-app.html` | GitHub App identity (manifest flow) | n/a — you create the App |
| `org/harden.sh` | Org security hardening (run after scope refresh) | n/a — run locally |

## TWO actions only an org owner can do (the remaining blockers)

### A. Grant `admin:org` scope (unblocks org hardening, check #2)
The active `gh` token (`zircote`) lacks `admin:org`, so org policy/Actions
writes return 403. Grant it once in this session:

```
! gh auth refresh -h github.com -s admin:org
```

Then I (or you) run `bash org/harden.sh`.

### B. Create the GitHub App (the automation identity, check #1)
GitHub has no API to create an App from scratch with a PAT — use the manifest flow:

1. `open app/create-app.html` (signed in as an attested-delivery owner).
2. Click "Create attested-delivery-ci App" → confirm on GitHub.
3. Copy the `?code=` from the redirect URL, then:
   `gh api -X POST /app-manifests/<code>/conversions --jq '{id, slug, html_url}'`
4. Save the returned `pem` OUTSIDE any repo (never commit it).
5. Open `html_url` → Install → "All repositories".

The App is the org automation identity (repo provisioning, agentic workflows,
cross-repo dispatch). It is **not** used for artifact signing or GHCR: in-workflow
signing uses the run's own ephemeral `GITHUB_TOKEN` + OIDC `id-token`, which is
what SLSA L3 requires.

## Goal check status

1. **App identity** — App half: manifest ready, pending your creation (B).
   No-PAT half: DONE — zero `secrets.GITHUB_PAT` across all workflows.
2. **Org hardening** — script ready (`org/harden.sh`), pending scope refresh (A).
   (2FA enforcement is a UI-only setting — see the script's note.)
3. **Templates** — DONE: `.github`, `attested-pipeline-template`, `rust-template`
   created public with `isTemplate: true`.
4. **Attested pipelines** — workflows SHA-pinned, pin-check present, provenance +
   SBOM attest + fail-closed verify configured; `actionlint` exit 0. Live
   `gh attestation verify` exercised via release dry-run.
5. **Docs site** — DONE: `pnpm build` exits 0, 26 pages, sitemap generated.
