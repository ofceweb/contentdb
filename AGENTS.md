# AGENTS.md

`contentdb` is the declaration log of the content database (OFCE, CEPII, IFE). It holds data and one workflow, and **no logic**: the checks live in the R package `ofceweb` (`R/check_registry.R`, schema `inst/schema/declaration-1.0.json`). See `README.md` for the declaration format, in French.

## Where things live

- `registry/{annee}/{id}.json`: one declaration per content. `{annee}` is the year of first publication and never changes.
- `.github/workflows/check-registry.yml`: on every push and pull request, lists files deleted from `registry/` (`git diff --no-renames --diff-filter=D`) and calls `ofceweb::check_registry_ci()`.
- Never committed here: the aggregated base `contents.json` and `collector-state.json` (published on the FTP server by the collection).

## Rules

- A file in `registry/` is never deleted or moved. A new declaration of the same content edits its file (`status`, `declared_at`, `declared_by`, or `manifest_url` for a new version); Git history is the log.
- A direct push to `main` is allowed: the check runs after the fact and a bad declaration is fixed by a new commit. GitHub cannot reject a push based on a workflow.
- A rule change goes into `ofceweb` (function, schema, examples, tests), then into `README.md` here.

## Commands

- Check locally: `Rscript -e 'ofceweb::check_registry(".")'` (needs `ofceweb` and `jsonvalidate`).
