# Decisions

This page lists the decisions that a maintainer must know. Each entry gives the decision, the reason, the version, and the place in the code.
The workflow builder and its gotchas are in [CLAUDE.md](../CLAUDE.md), which is not in git yet. This page does not repeat them. The house style of the package family is in Leela's CLAUDE.md. The changes of each release are in the [CHANGELOG](../CHANGELOG.md). Old design notes are in `docs/superpowers/`, which is not in git.

## What lives here, what does not

- **gheasy is the git plumbing of the family.** `gheasy.repo` runs git for one repository. `gheasy.repos` finds and lists a set of repositories. `gheasy.core` and `gheasy.workflow` set up GitHub repos and write CI workflows. shalya, ramabana and Leela import `gheasy.repo` by name. (0.0.8) `nbs/02_repo.ipynb`, `nbs/03_repos.ipynb`.
- **The repository code moved here from `ramabana.git`.** Do not add a second copy of git calls to an app. Patch gheasy. (0.0.8) `GitRepo`, `GitGateway`.
- **Helpers that Leela imports have a public name.** (2 Sep 2026)

## Running git

- **One gateway runs every git command.** `GitGateway` keeps a `RepoLock` for each repository. Read commands run at the same time, a write takes an exclusive lock, and a network command takes its own lock. The git calls of `gheasy.core` use the same gateway. (0.0.8) `RepoLock`, `GitGateway` in `nbs/02_repo.ipynb`.
- **Git and gh get `/dev/null` as stdin.** A command must not read or change the terminal of the caller. `gh secret set` and `gh variable set` are the exception, because gh asks for an empty value. (0.0.14) `GitGateway.run`.
- **A commit may wait 15 minutes.** A pre-commit hook such as `nbdev-prepare` runs the tests, so the 30 s limit of other commands is too short. (0.0.19) `HOOK_WAIT`.
- **Remote commands drop a stale token.** When gh has a keyring login for github.com, `clone`, `fetch`, `ls-remote`, `push` and `pull` run without `GITHUB_TOKEN` and `GH_TOKEN`. An old token in the shell then cannot stop a push. `os.environ` does not change. (0.0.18) `net_env`, `gh_keyring`.
- **`GitRepo.info` runs on a shared pool of four threads.** It runs serially when the thread holds the write lock or is a pool worker, so it cannot deadlock. An answer that began before `invalidate` is not cached. (0.0.11) `_pool`, `info`.

## Changing history

- **Each change records a safepoint first.** A safepoint is a ref under `refs/leela/safepoint` and a line in a journal. `undo` goes back to it. A harness keeps the undo token. (0.0.8, 0.0.15) `Safepoint`, `safepoint`, `undo`.
- **A preview rehearses a merge or a rebase without touching the worktree.** It replays every commit on the branch, not only the first 250. (0.0.15) `divergence` and the rehearsal in `nbs/02_repo.ipynb`.
- **A stop on conflicts keeps the work.** Uncommitted work is kept as a commit, and the note names it. A merge never resets away a conflict. (0.0.15)

## GitHub and CI

- **Workflow YAML comes from ruamel.yaml, not PyYAML.** GitHub Actions needs indented list items under `steps:`. See CLAUDE.md. `nbs/01_workflow.ipynb`.
- **gheasy needs ghapi 2.** The calls with `sync=True` use it, because ghapi is async. (0.0.7, 0.0.20) `pyproject.toml`.
- **A new repo of an organisation goes in the organisation.** (0.0.12) `gh_new`, `GheasyRepo.create`.
- **`gh_repo_info` tries once more with no token on a 401.** An old `GITHUB_TOKEN` must not hide a public repo. (0.0.20) `gh_repo_info` in `nbs/00_core.ipynb`.
