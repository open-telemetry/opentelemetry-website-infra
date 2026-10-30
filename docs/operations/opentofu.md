# OpenTofu operations

**Status: planned.** This document records the agreed operating model. The workflows and PR-comment commands described here are not implemented yet.

## Operating model

Use one production environment in the existing OpenTelemetry Cloudflare account. Run OpenTofu through GitHub Actions, including formatting, validation, imports, and provider lock-file updates. Contributors edit configuration and review workflow results in GitHub. Application checks can run locally before deployment.

Build one custom Collector with the OpenTelemetry Collector Builder (OCB). Each Stack uses this shared Collector. See the [project terminology](../CONTEXT.md) for these terms.

## Change workflow

1. Submit configuration changes in a PR. Fork contributions run checks without infrastructure credentials. Maintainers move reviewed changes to a repository branch for a credentialed plan.
2. An authorized maintainer can comment `tofu plan` to request a preview of changes on the repository branch.
3. After each relevant merge, GitHub Actions generates a fresh plan from `main` and posts a summary with its source revision on the PR.
4. A maintainer reviews the saved post-merge plan and comments `tofu apply` on the merged PR to approve that plan. The workflow applies it and reports the result.

Applies run after merge, one at a time for each state. Bind each approval to the plan available when the comment was created and verify that it targets the current `main` revision. A newer revision requires a new plan and approval; a replacement plan also needs new approval. Apply the approved saved plan without generating a replacement during the apply run.

After merge, `tofu plan` can also refresh the deployment plan. Scheduled drift checks report changes for maintainer review.

## State and ownership

Store OpenTofu state in a private Cloudflare R2 bucket through the S3 backend with native lockfiles. Keep state, saved plans, and backups private. Verify locking and state restoration through GitHub Actions before the first account apply.

Create the initial R2 bucket and account-owned credentials once through the Cloudflare dashboard, using the required administrator access. Keep public bucket access disabled and supply credentials through GitHub Actions secrets. Scope credentials to the resources and operations they need, and keep their values out of Git.

This repository's maintainers own credential rotation, state backups, and recovery from failed or interrupted runs. Document backup retention and recovery steps as part of the implementation. Recovery must work through the dashboard and automation without local OpenTofu commands.

## Planned repository layout

Add the directories as their components are implemented.

| Path | Responsibility |
|---|---|
| `.github/workflows/` | OpenTofu checks, plans, applies, and drift reporting; Collector builds and releases |
| `collector/` | OCB manifest, portable Collector image, shared configuration, and Collector tests |
| `infra/opentofu/` | Account and infrastructure resources, provider configuration, and state configuration |
| `infra/worker/` | Cloudflare Worker and its integration with the Collector container |
| `stacks/<backend>/` | Runtime configuration and deployment files for a Backend and its supporting services |
| `tests/smoke/` | Checks for the local and deployed Ingest endpoint |
| `docs/operations/` | Maintainer instructions for infrastructure operations and recovery |

Keep OpenTofu resource declarations under `infra/opentofu/`; they can reference runtime configuration under `stacks/`. Use one tool to manage each resource. OpenTofu manages account resources, while Wrangler handles Collector image uploads and Worker deployment.

Prometheus and Jaeger are examples of the planned Stack layout. Their hosting and persistence will be defined when those components are implemented.
