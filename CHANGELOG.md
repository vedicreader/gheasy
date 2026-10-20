# Release notes

<!-- do not remove -->

## 0.0.14
- `GitGateway.run` gives Git `/dev/null` as stdin when no `input` is passed, so a Git command never reads or reconfigures the caller's terminal. The ripgrep scan in `find_repos`, `gh_lfs`, `GheasyRepo.new` and `WorkflowDoc.watch`/`logs` do the same. `gh secret set` and `gh variable set` keep the terminal, since an empty value makes `gh` ask for one.

## 0.0.13
- `mk_dependabot` keeps `labels` when `assignees` is set, and writes plain YAML lists.
- `rust_job` and `go_job` name their jobs `rust` and `go`, so one workflow can hold both.
- The docs drop an example that called `gh_pr` and `gh_pr_merge`, which gheasy does not have.

## 0.0.12
- `gh_new` / `GheasyRepo.create` make an org's repo in the org; they made it under the signed-in user.
- `gh_workflow` takes the list of workflow names `gh_init` stores.

## 0.0.11
`GitRepo.info` runs its git commands on a shared pool of four threads. It runs them serially when the calling thread holds the repository's write lock or is itself a pool worker. A failure raises the same error the serial order raised first. `invalidate` now starts a new cache generation, and an answer begun before it is not cached. `info` seeds the `decorations` cache only when it computed a new answer. The git directory is looked up once per repository and cached for 60 seconds.

## 0.0.10
repos management for leela

## 0.0.9
release

## 0.0.8

New `gheasy.repo`: git repository operations, moved here from `ramabana.git`. One gateway that
serialises every git process per repository, a safepoint before every mutation, and previews that
rehearse a merge or a rebase without touching the worktree. `gheasy.core`'s own git calls now go
through the same gateway.


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
