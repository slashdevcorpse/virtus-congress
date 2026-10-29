# virtus-congress

This repo is the Virtus fork of the upstream project. Virtus uses it as a Python wheel for the federal bills family
(B1).

- Upstream: https://github.com/unitedstates/congress (licence CC0-1.0).
- Base: upstream `main` at `8184bcb13160da2389dea4c902de67817db681fc`.

## Branches

- `main` is equal to upstream `main`. Only a fast-forward from upstream moves it.
- `virtus` holds our patches as separate commits on top of an upstream commit.

## Patches

One line for each commit on `virtus` that is not in upstream:

1. `Build the wheel without the congress-legislators checkout`: `setup.py` includes only the `congress` package, so a
   `congress-legislators` checkout in the tree does not go into the wheel or the sdist. `.gitmodules` is removed
   (the tree has no gitlink, and the `git://` URL does not work). The tasks that need the legislator data still
   clone it at run time (`congress/tasks/utils.py`, `require_congress_legislators_repo`). This commit also added a
   `pyproject.toml` with a build-backend pin; patch 5 removes it.
2. `Release workflow: wheel, sdist and SHA256SUMS for v*+virtus.* tags`: `.github/workflows/virtus-release.yml`, and
   the version `0.0.1+virtus.1` in `setup.py`.
3. `VIRTUS.md: upstream, patches, release and sync steps`: this file.
4. `Release workflow: fail the asset check when an archive can't be listed`: the check lists each archive before
   `grep` reads it, so a failed listing stops the step.
5. `Pin setuptools in the release workflow, not in pyproject.toml`: the release build pins the build backend
   (`setuptools==84.2.1`) with a build constraint. The repo has no `pyproject.toml`, so `pip install .` works on the
   Pythons upstream supports.
6. `VIRTUS.md: merge upstream/main in the sync steps`: the sync steps merge `upstream/main`, not a local `main`
   that can be old.
7. `VIRTUS.md: list every patch commit`: this list names each commit.

## Python 3.13

No patch is necessary. Python 3.13.15 and Python 3.10 give the same results. These upstream problems occur on
both versions:

- `congress.tasks.statutes` imports `bill_versions`, which upstream does not have.
- `congress.tasks.voteview` opens `tasks/voteview_codedoptions.csv` relative to the current directory when it is
  imported. The wheel does not hold this file.
- `congress.contrib.beanstalkd` needs `beanstalkc`, which is not a dependency.
- `test/run` has one failure, `test_reported_from_committee`.
- Some regular expressions use invalid escape sequences. Python 3.13 shows a `SyntaxWarning` when it compiles them.

## Release

1. Increase `<n>` in the `setup.py` version `<upstream version>+virtus.<n>`.
2. Tag the `virtus` commit `v<upstream version>+virtus.<n>` and push the tag.
3. The workflow `virtus-release` builds the wheel and the sdist with `uv build` and publishes them with one
   `SHA256SUMS` as the release assets of the tag.
4. In `virtus`, run `tools/pins/bump.py virtus-congress <tag>`.

## Sync with upstream

1. Add the upstream remote once: `git remote add upstream https://github.com/unitedstates/congress`.
2. Fast-forward `main` to upstream `main`: `git fetch upstream && git push origin upstream/main:main`.
3. Merge `upstream/main` into `virtus` (never rebase `virtus`). Solve the conflicts in the patch commits.
4. Update "Base" in this file. If the upstream version changes, set the `setup.py` version to
   `<new upstream version>+virtus.1`.
5. Do a release.
