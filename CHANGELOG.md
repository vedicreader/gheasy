# Release notes

<!-- do not remove -->

## 0.0.10

Six helpers Leela was importing through the underscore are public and exported. `gheasy.repo` gains
`invalidate`, `plural` and `unborn`; `gheasy.core` gains `gh_api` and `gh_token`; `gheasy.workflow`
gains `yaml_instance`. The private spellings are gone: an embedder reading what gheasy wrote needs
the same ruamel settings and the same cache invalidation, and neither was reachable by name.


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
