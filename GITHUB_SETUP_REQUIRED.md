# Required fields before the first public push

This repository is structurally prepared for GitHub, but author identity and the
remote repository address must come from the repository owner. Replace every
placeholder below before committing.

| Placeholder | File | Replace with |
| --- | --- | --- |
| `YOUR_GIVEN_NAME` | `DESCRIPTION`, `CITATION.cff`, `.zenodo.json.template` | Given name of the package author or maintainer |
| `YOUR_FAMILY_NAME` | same files | Family name of the package author or maintainer |
| `YOUR_EMAIL@example.org` | `DESCRIPTION` | Public maintainer email |
| `YOUR_ORCID` | `CITATION.cff`, `.zenodo.json.template` | ORCID in `0000-0000-0000-0000` form; the setup script removes the field if unavailable |
| `YOUR_AFFILIATION` | `.zenodo.json.template` | Institutional affiliation |
| `YOUR_GITHUB_USERNAME` | `DESCRIPTION`, `README.md`, `README_CN.md`, `GITHUB_RELEASE_GUIDE_CN.md` | GitHub account or organization name |
| `YOUR_COPYRIGHT_HOLDER` | `LICENSE`, `LICENSE.md` | Legal copyright holder |

Run the preflight script after replacement:

```powershell
powershell -ExecutionPolicy Bypass -File tools/preflight_github.ps1
```

The script checks identity placeholders, private or oversized data files,
machine-local secrets, package metadata, and the Git working tree. It does not
upload anything.
