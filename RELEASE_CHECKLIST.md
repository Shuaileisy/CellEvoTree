# CellEvoTree 0.1.0 Release Checklist

## Required identity fields

- [ ] Replace `YOUR_GIVEN_NAME` and `YOUR_FAMILY_NAME` in `DESCRIPTION`.
- [ ] Replace `YOUR_EMAIL@example.org` with the maintainer email.
- [ ] Replace `YOUR_GITHUB_USERNAME` in `DESCRIPTION`, `README.md`, and release instructions.
- [ ] Add ORCID identifiers where available.
- [ ] Replace the copyright holder in `LICENSE`.
- [ ] Replace the copyright holder in `LICENSE.md`.
- [ ] Replace the placeholder author in `CITATION.cff`.
- [ ] Add the final GitHub `URL` and `BugReports` fields to `DESCRIPTION`.
- [ ] Fill `.zenodo.json.template` after the author order is final.
- [ ] Run `powershell -ExecutionPolicy Bypass -File tools/preflight_github.ps1` and resolve every failure.

## Repository checks

- [ ] Confirm the repository visibility: private or public.
- [ ] Confirm that `benchmarks/` remains excluded from Git.
- [ ] Confirm that no `.RData`, `.rds`, `.h5ad`, `.loom`, `.mtx`, credentials,
      or patient-identifiable files are staged.
- [ ] Run `git diff --cached` before the first commit.
- [ ] Run local package build and check.
- [ ] Confirm GitHub Actions passes on Windows, macOS, and Ubuntu.

## Release

- [ ] Create the initial commit on `main`.
- [ ] Create tag `v0.1.0`.
- [ ] Create a GitHub Release from the tag.
- [ ] Upload `CellEvoTree_0.1.0.tar.gz`.
- [ ] Test installation from GitHub in a clean R library.
