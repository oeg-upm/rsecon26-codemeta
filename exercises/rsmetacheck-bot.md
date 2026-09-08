# Workshop Exercise: Reproduce the rsmetacheck-bot Workflow

> A hands-on exercise that walks a participant through installing, configuring,
> running, and publishing results with **rsmetacheck-bot**, an automated bot that analyzes repository metadata
> quality and opens improvement issues on GitHub/GitLab.

---

## 1. Context & Purpose

**rsmetacheck-bot** is part of the
[CodeMetaSoft](https://w3id.org/codemetasoft) initiative to improve the quality
of research-software metadata. The bot:

1. Analyzes one or more software repositories.
2. Detects metadata *pitfalls* (critical issues) and *warnings* (recommended
   improvements) using the [rsmetaCheck](https://github.com/SoftwareUnderstanding/rsmetaCheck) engine.
3. Generates a set of machine-readable and human-readable reports.
4. Optionally opens, updates, or closes issues on the target repositories.

The bot **never** modifies your code, never opens pull requests, and never has
access to your repository secrets. Its only write action is the (optional)
creation/update/closure of issues.

### Learning objectives

By the end of this exercise, participants will be able to:

- Install the tool and its dependencies.
- Configure authentication tokens for GitHub and/or GitLab.
- Author a single JSON configuration file that drives a whole analysis campaign.
- Run a **dry-run analysis** and interpret every output artifact.
- Safely **simulate** publishing, then **publish** real issues.
- Summarize a campaign into aggregate statistics for a report or presentation.

---

## 2. Prerequisites

| Requirement | Why | How to check |
|---|---|---|
| Python 3.11 or 3.12 | Runtime | `python --version` |
| Git | Commit lookup (`git ls-remote` fallback) | `git --version` |
| `uv` (recommended) or `pip` | Dependency management | `uv --version` |
| A GitHub **and/or** GitLab account | Issue creation | log in to the platform |
| A personal access token (PAT) | API authentication | see [Step 2](#step-2--configure-authentication) |
| One repository you own | Safe target for the exercise | create one if needed |

> **Important:** use a repository **you own** (or a fork) so you have the
> `issues: write` permission required to publish issues. Never run the publish
> step against repositories you do not control.

---

## 3. Before You Start: Prepare a Target Repository

The most instructive experience comes from a repository with *intentionally
incomplete metadata*, so the bot has something to find. We will be cloning this repository: https://github.com/SoftwareUnderstanding/Metadata-Adoption-Quantify.

---

## 4. Step 1 — Install the Tool

### Option A: Install from PyPI (recommended for participants)

```bash
# with uv
uv add sw-metadata-bot

# or with pip
pip install sw-metadata-bot
```

### Option B: Install from source (recommended for maintainers/contributors)

```bash
git clone https://github.com/SoftwareUnderstanding/sw-metadata-bot.git
cd sw-metadata-bot
uv sync
```

When installing from source, prefix every command with `uv run`, e.g.:

```bash
uv run sw-metadata-bot --help
```

### Verify the installation

```bash
sw-metadata-bot --help
```

Expected output lists the available subcommands:

```
Commands:
  run-analysis       Run analysis and compute issue lifecycle decisions (dry-run).
  publish            Publish issues using precomputed decisions.
  simulate-publish   Simulate publish using a local fake issue client.
  verify-tokens      Verify that tokens are set and have correct permissions.
  report-summary     Summarize report.json and save a summary file.
```

---

## 5. Step 2 — Configure Authentication

The bot reads tokens from environment variables:

| Variable | Platform | Purpose |
|---|---|---|
| `GITHUB_API_TOKEN` | GitHub.com | Create/update/close issues |
| `GITLAB_API_TOKEN` | GitLab.com | Create/update/close issues |

### Create the tokens

- **GitHub:** Settings → Developer settings → Personal access tokens → Tokens
  (classic) or fine-grained. Grant the **`issues: write`** scope (and
  `contents: read` to read metadata).
- **GitLab:** Preferences → Access Tokens. Grant the **`api`** scope.

### Export the tokens

```bash
export GITHUB_API_TOKEN=ghp_xxxxxxxxxxxx
export GITLAB_API_TOKEN=glpat_xxxxxxxxxxxx
```

### (Recommended) Use a `.env` file

Create a `.env` file in the working directory:

```bash
GITHUB_API_TOKEN=ghp_xxxxxxxxxxxx
GITLAB_API_TOKEN=glpat_xxxxxxxxxxxx
```

Load it in your shell before running commands:

```bash
set -a; source .env; set +a
```

> The `verify-tokens` and `publish` commands auto-load a `.env` file if one
> exists in the current directory.

---

## 6. Step 3 — Create the Configuration File

The entire pipeline is **config driven**. A single JSON file defines:

- which repositories to analyze,
- the custom message appended to issues,
- inline opt-outs (analyze but never publish),
- where outputs are written.

Create `config.json`:

```json
{
  "version": "1.1.0",
  "analysis": {
    "repositories": [
      "https://github.com/SoftwareUnderstanding/Metadata-Adoption-Quantify"
    ],
    "generate_codemeta_if_missing": true
  },
  "issues": {
    "custom_issue_message": "Your repository was analyzed as part of the metadata quality workshop. Several metadata issues were identified and could be addressed.",
    "opt_outs": []
  },
  "outputs": {
    "output_root_dir": "outputs",
    "run_name": "workshop_run",
    "snapshot_tag_format": "%Y%m%d"
  }
}
```

### Configuration reference

| Field | Required | Description |
|---|---|---|
| `version` | no | Config schema version (default `"1.1.0"`). |
| `analysis.repositories` | **yes** | List of repository URLs to analyze. |
| `analysis.generate_codemeta_if_missing` | no | If `true`, the bot generates a `codemeta.json` when missing (default `true`). |
| `analysis.rsmetacheck_config_file` | no | Path to a custom RSMetaCheck TOML config. |
| `analysis.rsmetacheck_config_profile` | no | Named profile in the RSMetaCheck config. |
| `issues.custom_issue_message` | no | Custom message appended to generated issue bodies. |
| `issues.opt_outs` | no | Repositories that are analyzed but for which **no issue is published**. |
| `outputs.output_root_dir` | no | Root output directory (default `outputs`). |
| `outputs.run_name` | no | Stable campaign name used as parent folder for snapshots. |
| `outputs.snapshot_tag_format` | no | `strftime` pattern for snapshot folders (default `%Y%m%d`). |

> **Supported platforms:** GitHub.com, GitLab.com, and self-hosted GitLab
> instances (the latter requires a token for the target host).

---

## 7. Step 4 — Verify Tokens

Before running, confirm your tokens are valid and have the right permissions:

```bash
sw-metadata-bot verify-tokens
```

Or scope it to a single platform, or request JSON output:

```bash
sw-metadata-bot verify-tokens --github
sw-metadata-bot verify-tokens --gitlab --json
```

**What to expect:** a table per platform showing:

- `Token set` ✅/❌
- `Authenticated` ✅/❌
- the authenticated `User`
- `Scopes`
- `Issues permission` ✅/⚠️
- `Contents permission` ✅/⚠️

If a token is missing or lacks the issues permission, fix it before continuing.

---

## 8. Step 5 — Run the Analysis (Dry-Run)

The `run-analysis` command **only analyzes**. It computes issue lifecycle
decisions and writes reports, but does **not** contact the GitHub/GitLab API
for writes. This is the safe "dry-run" stage.

```bash
sw-metadata-bot run-analysis --config-file config.json
```

(From a source checkout, use `uv run sw-metadata-bot run-analysis --config-file config.json`.)

### Useful options

| Option | Description |
|---|---|
| `--config-file` | **(required)** path to the JSON configuration file. |
| `--snapshot-tag <tag>` | Override the snapshot folder name (e.g. `2026-03`). |
| `--previous-report <path>` | Previous `run_report.json` for incremental issue handling. |
| `--force-analysis` | Re-analyze even if the repository commit is unchanged. |

### What happens under the hood

For each repository the bot:

1. Resolves the repository's current HEAD commit (via `git ls-remote`).
2. Compares it to the previous snapshot (incremental reuse when unchanged).
3. Runs RSMetaCheck (SOMEF metadata extraction → pitfall/warning detection).
4. Optionally generates a `codemeta.json` when one is missing.
5. Computes a lifecycle decision (`created`, `updated_by_comment`, `closed`,
   `skipped`, or `failed`).

---

## 9. Step 6 — Inspect the Outputs (What to Expect)

Given the config above, `run-analysis` produces the following layout:

```text
outputs/
└── workshop_run/
    └── 20260908/
        ├── config.json
        ├── analysis_results.json
        ├── run_report.json
        └── github_com_<you>_my_metadata_test/
            ├── issue_report.md
            ├── pitfall.jsonld
            ├── report.json
            └── somef_output.json
```

### Per-snapshot files

| File | Description |
|---|---|
| `config.json` | Snapshot of the effective configuration (for reproducibility). |
| `analysis_results.json` | Global summary: evaluated repositories + commit IDs. |
| `run_report.json` | Top-level decision report: one record per repository. |

### Per-repository files

| File | Description |
|---|---|
| `issue_report.md` | **Human-readable markdown report** — the issue body you can review before publishing. |
| `pitfall.jsonld` | Raw JSON-LD output from RSMetaCheck (detected checks + evidence). |
| `report.json` | Machine-readable per-repo report: findings, decision/action, identifiers, links. |
| `somef_output.json` | Raw SOMEF metadata extraction used as analysis input. |

> Repository folder names are the sanitized, lower-cased host + path, e.g.
> `https://github.com/SoftwareUnderstanding/sw-metadata-bot` →
> `github_com_softwareunderstanding_sw_metadata_bot`.

### The `run_report.json` record

Each record describes the lifecycle decision for one repository and includes
fields such as:

- `repo_url` — the repository URL.
- `action` — one of `created`, `updated_by_comment`, `closed`, `skipped`,
  `failed`, or `simulated_created` (dry-run mode).
- `reason_code` — why the action was taken (e.g. `unsubscribe`,
  `commit_unchanged`, `opt_out`, `exception`).
- `pitfalls_count` / `warnings_count` — number of detected checks.
- `pitfalls_ids` / `warnings_ids` — the specific check codes.
- `current_commit_id` / `previous_commit_id`.
- `dry_run` — `true` in analysis mode.

### Example `issue_report.md` content

```markdown
### [P002](https://w3id.org/rsmetacheck/catalog/#P002)
**Evidence:** P002 detected: LICENSE file contains unreplaced template placeholders

**Suggestion:** Update the copyright section with accurate names, organizations, and the current year. Personalizing this section ensures clarity and legal accuracy.
```

---

## 10. Interpreting the Checks (Pitfalls & Warnings)

RSMetaCheck check codes follow a convention: `P####` = pitfall (data-quality
issue), `W####` = warning (best-practice issue). A selection:

| Code | Meaning |
|---|---|
| `P001` | Metadata version does not match the latest release. |
| `P002` | `LICENSE` contains unreplaced template placeholders. |
| `P003` | Multiple authors in a single field instead of a list. |
| `P004` | `codemeta.json` `README` points to homepage instead of README. |
| `P006` | License points to a local file instead of stating a name. |
| `P014` | Bare DOIs in `identifier` instead of `https://doi.org/...`. |
| `P017` | `codemeta.json` version does not match the package version. |
| `W001` | Software requirements lack version specifications. |
| `W002` | `dateModified` is outdated vs. the repository's last update. |
| `W004` | Programming languages listed without versions. |

Full catalog: https://w3id.org/rsmetacheck/catalog/

---

## 11. Step 7 — Simulate Publishing (Safe)

Before writing anything to a real platform, use the local fake client:

```bash
sw-metadata-bot simulate-publish --analysis-root outputs/workshop_run/<snapshot>
```

Options:

| Option | Description |
|---|---|
| `--analysis-root` | **(required)** snapshot folder containing `run_report.json`. |
| `--retry-failed` | Retry eligible failed records. |
| `--unsubscribe` | Simulate an `unsubscribe` comment on all issue-comment checks. |
| `--fake-comment <text>` | Return a fake comment for all issue URLs (repeatable). |

**What to expect:** the same decisions are applied against an in-memory fake
client, so no real issues are created. Use it to verify the decision tree before
publishing.

---

## 12. Step 8 — Publish Real Issues

Once you are satisfied with the generated reports:

```bash
sw-metadata-bot publish --analysis-root outputs/workshop_run/<snapshot>
```

**What happens:**

- `created` → a new issue is opened on the repository.
- `updated_by_comment` → a comment is added to an existing issue.
- `closed` → the existing issue is closed (no findings remain).
- `skipped` → no write action (e.g. already published, opted out, unsubscribed).

### Retrying transient failures

```bash
sw-metadata-bot publish \
  --analysis-root outputs/workshop_run/<snapshot> \
  --retry-failed
```

Already-posted records remain idempotent (skipped); only eligible failed
records are re-attempted.

---

## 13. Step 9 — Summarize the Campaign

Generate aggregate statistics for a report or presentation:

```bash
sw-metadata-bot report-summary --analysis-root outputs/workshop_run/<snapshot>
```

This writes `report_summary.json` into the snapshot folder, containing:

- `repository_count`
- `total_pitfalls`, `total_warnings`
- `pitfalls_by_id`, `warnings_by_id` (histograms of check codes)
- `actions` and `reason_codes` (decision breakdown)
- `reason_codes_by_action` and `reason_code_percentages_by_action`
- `pitfalls_per_repository`, `warnings_per_repository`
- `issues_created` and `issues_created_per_repository`
- `tool_metadata` and `run_metadata`

These numbers are ideal for a slide deck: "Analyzed N repositories, detected X
pitfalls and Y warnings, opened Z issues."

---


### Handling `unsubscribe`

If a repository maintainer comments **`unsubscribe`** on an issue, the bot
detects it during publish, adds the repository to the config's `opt_outs`, and
stops publishing issues for it (the repository is still analyzed).

---

## 14. Full Workshop Cheat Sheet

```bash
# 0. Install
pip install sw-metadata-bot                 # or: uv add sw-metadata-bot

# 1. Configure tokens
export GITHUB_API_TOKEN=ghp_xxxxxxxxxxxx
export GITLAB_API_TOKEN=glpat_xxxxxxxxxxxx
sw-metadata-bot verify-tokens

# 2. Write config.json (see Step 3)

# 3. Run analysis (dry-run, no writes)
sw-metadata-bot run-analysis --config-file config.json

# 4. Inspect outputs
ls -R outputs/workshop_run/<snapshot>/
cat outputs/workshop_run/<snapshot>/run_report.json
cat outputs/workshop_run/<snapshot>/<repo_folder>/issue_report.md

# 5. Simulate publish (safe)
sw-metadata-bot simulate-publish --analysis-root outputs/workshop_run/<snapshot>

# 6. Publish real issues
sw-metadata-bot publish --analysis-root outputs/workshop_run/<snapshot>

# 7. Summarize for reporting
sw-metadata-bot report-summary --analysis-root outputs/workshop_run/<snapshot>
```

---

## 15. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `401` / auth failed | Missing/invalid token | Re-export `GITHUB_API_TOKEN`/`GITLAB_API_TOKEN`, run `verify-tokens`. |
| `403`/`404` on issue creation | Insufficient repo permissions | Use a repo you own; grant `issues: write`. |
| Platform not supported | Non-GitHub/GitLab URL | Use GitHub.com, GitLab.com, or self-hosted GitLab. |
| No issues published | Repo in `opt_outs`, unchanged commit, or `unsubscribe` | Check `reason_code` in `run_report.json`. |
| Publish failed (transient) | Rate limit / timeout / 5xx | Rerun with `--retry-failed`. |
| Analysis reused old results | Commit unchanged | Re-run with `--force-analysis`. |

---