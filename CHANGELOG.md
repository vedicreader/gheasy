# Release notes

<!-- do not remove -->

## 0.0.19
- `GitRepo.commit` waits up to `HOOK_WAIT` (900 s) for the commit. Before, a commit had the 30 s limit of other git commands, so a pre-commit hook that runs the tests, such as `nbdev-prepare`, made each commit fail with `timed out after 30s`. Other commands keep their limits.

## 0.0.18
- Git commands that talk to a remote (`clone`, `fetch`, `ls-remote`, `push` and `pull`) run without `GITHUB_TOKEN` and `GH_TOKEN` when `gh` has a keyring login for github.com. The `gh auth git-credential` helper then uses the keyring login. Before, a token that was not valid any more stopped `GitRepo.push` with `Invalid username or token`. gheasy asks `gh auth status` once and keeps the answer for five minutes. With no token variable set, or with no keyring login, nothing changes. `os.environ` never changes. The new `GitGateway.net_env` and `GitGateway.gh_keyring` do this work.

## 0.0.17
gh create of an existing remote does not error out

## 0.0.15
- `GitRepo.undo` reports `restored` as the number of tracked files in the safepoint's stash. It reported the number of uncommitted files, which includes untracked files that the stash does not hold.
- `gh_pyproject_to_hatchling` writes `pyproject.toml` and returns `True` when `requires-python` is the only value that changes. It left the file unchanged in that case.
- `gh_secret`, `gh_push_env`, `gh_secrets_from_file` and `gh_deploy_key_setup` take `path` and run `gh` in that repository.
- `mk_workflow` adds `needs='test'` to the `publish` and docker jobs only when the workflow has a `test` job. `mk_deploy_job` does the same.
- `uv_pypi_job` adds the `release` trigger with type `published` when the workflow has no `release` trigger.
- `uv_test_job` passes the matrix Python version to `setup_uv`, which takes a new `python_version` argument.
- `StrategyBuilder.include` and `exclude` write hyphenated keys, and a strategy with only `include` or `exclude` entries renders.
- `WorkflowDoc.dispatch`, `upload` and `list_runs` call GitHub synchronously.
- The pull request check list reads every page of check runs.
- `GitRepo.divergence` counts commits with `git rev-list --count`, and the rebase rehearsal replays every commit on the branch. It stopped at 250.
- `GitRepo` previews count untracked files separately from blocking changes. An operation that stops on conflicts keeps your uncommitted work as a commit and names it in the note.
- `GheasyRepo.new` no longer passes a token to `gh_workflow`.

## 0.0.14
- `GitGateway.run` passes `/dev/null` as Git's stdin when no `input` is given. A Git command no longer reads from or reconfigures the caller's terminal. The ripgrep scan in `find_repos`, `gh_lfs`, `GheasyRepo.new` and `WorkflowDoc.watch`/`logs` do the same. `gh secret set` and `gh variable set` keep the terminal, because `gh` asks for a value when the value is empty.

## 0.0.13
- `mk_dependabot` keeps `labels` when `assignees` is set, and writes plain YAML lists.
- `rust_job` and `go_job` name their jobs `rust` and `go`. One workflow can hold both.
- The docs no longer include an example that called `gh_pr` and `gh_pr_merge`. gheasy does not have these functions.

## 0.0.12
- `gh_new` and `GheasyRepo.create` now create an organisation's repository in the organisation. They created it under the signed-in user.
- `gh_workflow` takes the list of workflow names `gh_init` stores.

## 0.0.11
`GitRepo.info` runs its git commands on a shared pool of four threads. It runs them one after another when the calling thread holds the write lock of the repository or is a pool worker. A failure raises the error that the serial order raised first. `invalidate` now starts a new cache generation. An answer that began before `invalidate` is not cached. `info` seeds the `decorations` cache only when it computed a new answer. The git directory is looked up once per repository and cached for 60 seconds.

## 0.0.10
repos management for leela

## 0.0.9
release

## 0.0.8

New `gheasy.repo`: git repository operations, moved here from `ramabana.git`. One gateway controls concurrent git commands for each repository. Read commands can run concurrently. Write commands take an exclusive lock. Network commands use a separate lock. A safepoint is recorded before every mutation.
Previews rehearse a merge or a rebase without touching the worktree. The git calls of `gheasy.core`
now use the same gateway.


## 0.0.7
ghapi is async, so sync=True for now



## 0.0.6
gheasy new setup python requires to >=3.11



## 0.0.5
pins python to 3.13, git workflows optional



## 0.0.4
nbdev pyproject to hatchling bugfix + cli addition



## 0.0.3
skills



## 0.0.2
gheasy makes git lfs, worfklows easy



## 0.0.1
initial release