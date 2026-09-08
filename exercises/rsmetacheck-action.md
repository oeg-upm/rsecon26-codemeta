# Hands-On Exercise: Validate Your Repository's Software Metadata with the RsMetaCheck GitHub Action

This document is a step-by-step exercise guide. Follow it end-to-end on one of your own
GitHub repositories to learn how to set up, run, and interpret the **RsMetaCheck** GitHub
Action, a tool that automatically detects metadata quality problems (pitfalls and warnings)
in software repositories using [SoMEF](https://github.com/KnowledgeCaptureAndDiscovery/somef).

---

## 1. Background

Software repositories should carry accurate, complete, machine-readable metadata so that
they can be discovered, cited, and reproduced. Common places for this metadata are:

- `codemeta.json`
- `package.json`, `pyproject.toml`, `setup.py`, `pom.xml`, `DESCRIPTION`, `CITATION.cff`
- `LICENSE`
- the repository's README

rsmetacheck analyzes this metadata and flags **29 categories of issues**, divided into:

- **Pitfalls (P001–P019):** genuine metadata defects (e.g. version mismatches, broken URLs,
  license template placeholders, bare DOIs).
- **Warnings (W001–W010):** lower-severity quality issues (e.g. dependencies without version
  constraints, an outdated `dateModified`, a missing identifier).

The GitHub Action wraps the [rsmetacheck Python tool](https://pypi.org/project/rsmetacheck/)
so the check runs automatically on every `push` and `pull_request`.

---

## 2. What you will learn

By the end of this exercise you will be able to:

1. Add the RsMetaCheck GitHub Action to a repository.
2. Understand what each input does and how to pass a GitHub token.
3. Trigger a run and navigate the GitHub Actions UI.
4. Read the **step summary**, **annotations**, and **outputs**.
5. Interpret the difference between pitfalls, warnings, evidence, and suggestions.

---

## 3. Prerequisites

- A GitHub account.
- A repository you own (a test repository is recommended if you do not want CI failures on a
  production project).
- Basic familiarity with editing files in the GitHub web interface or via `git`.

---

## 4. Setup — add the workflow file

### 4.1 Create the workflow directory and file

1. Open your repository on GitHub.
2. Click **Add file** -> **Create new file**.
3. In the "Name your file" field, enter:
   ```
   .github/workflows/rsmetacheck.yml
   ```
   (Creating the path automatically creates the `.github/workflows` directory.)
4. Paste the following content into the editor:

```yaml
name: RSMetaCheck Action

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  rsmetacheck:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Run RSMetaCheck
        uses: SoftwareUnderstanding/rs-metacheck-action@0.3.5
        with:
          verbose: "true"
          github_token: "${{ secrets.GITHUB_TOKEN }}"
```

5. Click **Commit changes** (commit directly to `main`).

> **Note on the `main` branch:** if your repository's default branch is named `master`,
> change both `branches: ["main"]` entries to `branches: ["master"]`, or set them to the
> branch you actually use.


---


## 5. How to run (trigger) the action

The action runs automatically. To observe a run:

1. After committing the workflow file, the initial `push` already triggers a run.
2. To trigger again, make any change (e.g. edit the README) and push, or open a pull request.
3. Go to the **Actions** tab of your repository.
4. Click the workflow run named **"RSMetaCheck Action"**.
5. Click the job **`rsmetacheck`** to expand the steps and watch the logs in real time.

In the logs you will see:

- `Executing RsMetaCheck command: ...`
- SoMEF running and extracting metadata from your repository.
- RSMetaCheck analyzing the extracted metadata.
- A list of detected codes, e.g. `P001 - Pitfall found in output_1.json`.
- `Generating GitHub Actions output...`

---

## 6. What to expect — outputs

### 6.1 Step summary

At the bottom of the workflow run page, a Markdown summary is rendered with these sections:

1. **Status table** — an overall status with totals:

   ```
   | Status | Pitfalls | Warnings |
   |--------|----------|----------|
   | ❌ **Pitfalls Detected** | 2 | 1 |
   ```

   The status cell is one of:
   - `❌ Pitfalls Detected` (at least one pitfall)
   - `⚠️ Warnings Detected` (warnings only)
   - `✅ Passed` (nothing detected)

2. **Repositories Analyzed** — the repository name and URL.

3. **Detected Pitfalls** — one row per pitfall code:

   ```
   | Code | Description | Evidence | Suggestion |
   |------|-------------|----------|------------|
   | P001 | ... | codemeta.json version '1.2.0' does not match release version '1.3.0' | Ensure the version ... matches the latest official release. |
   ```

4. **Detected Warnings** — same columns, one row per warning code.

5. **Per-Repository Details** — a collapsible section per repository showing every
   detected check with its evidence and a concrete suggestion.

### 6.2 Annotations

Detected issues also appear as annotations at the top of the run page (and inline on pull
request diffs):

- Pitfalls appear as red `::error::` annotations.
- Warnings appear as yellow `::warning::` annotations.
- Each annotation links to the RsMetaCheck catalog, e.g.
  `https://w3id.org/rsmetacheck/catalog/#P001`.

### 6.3 Exit status (pass/fail)

- If **any pitfall or warning** is found, the action exits with code `1`, so the step and
  job show as **failed** (red).
- If nothing is found, it exits `0` and the job passes (green).

> Because the step fails on findings, a real run on a typical repository will often show a
> red ✕. This is expected behavior, not a broken action.

### 6.4 Job outputs

The action exposes these outputs (usable by later steps with `${{ steps.<id>.outputs.X }}`):

| Output | Description |
|--------|-------------|
| `has_pitfalls` | `true` if any pitfall or warning was detected |
| `total_pitfalls` | Number of pitfalls (P-codes) |
| `total_warnings` | Number of warnings (W-codes) |
| `pitfalls_found` | JSON array of pitfall codes, e.g. `["P001","P003"]` |
| `warnings_found` | JSON array of warning codes, e.g. `["W007"]` |

### 6.5 Files produced in the workspace

These are created during the run (visible if you add an artifact-upload step):

- `somef_outputs/` — raw SoMEF extraction JSON files.
- `pitfalls_outputs/` — per-repository JSON-LD files (`*_pitfalls.jsonld`) containing the
  structured checks, evidence, and suggestions.
- `analysis_results.json` — the aggregated summary (counts, percentages, evaluated repos).

---

## 7. Interpreting the results

- **Pitfalls (P-codes)** are problems you should ideally fix. Read the **Suggestion** column
  in the summary (or the per-repo details) for a concrete remediation.
- **Warnings (W-codes)** are quality improvements; fix them when convenient.
- The **Evidence** column shows exactly what the tool observed (file + value), which is useful
  for verifying whether the detection is a true positive.
- Use the catalog link in each annotation for a detailed description of the check and how to
  fix it.

Example interpretation of a `P001` finding:

> *"The `codemeta.json` version is `1.2.0` but the latest release tag is `1.3.0`. Update the
> version in your metadata to `1.3.0` so it matches the official release."*

---


## 8. Troubleshooting

| Symptom | Likely cause / fix |
|---------|--------------------|
| Workflow never starts | Check the YAML indentation and the `on:` branch names match your default branch. |
| "input argument is required" error | The `GITHUB_REPOSITORY` context is unavailable; pass `input` explicitly. |
| SoMEF rate-limit errors | Add `github_token: "${{ secrets.GITHUB_TOKEN }}"`. |
| Long runtime | URL-validation pitfalls make network requests; this is normal. |
| No summary shown | The summary only appears when the step completes; open the run page and scroll to the bottom. |

---

