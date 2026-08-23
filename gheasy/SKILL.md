# gheasy

gheasy generates GitHub Actions workflows from Python,
routes secrets and variables from a local env into GitHub,
and audits repo health.

## Workflow builder

```python
from gheasy.workflow import Workflow

wfb = Workflow("ci")
wfb.on.push(branches=["main"]).pull_request()
wfb.uv_lint_job()
wfb.uv_test_job(needs="lint")
wfb.uv_pypi_job(needs="test")
print(wfb.build().to_yaml())
```

The `uv_*_job` presets bundle the usual checkout, setup-uv, install,
and run steps. For anything else, build a job step by step:

```python
wfb.job("deploy").needs("test").runs_on("ubuntu-latest")\
    .checkout().end_step()\
    .setup_uv().end_step()\
    .step("Deploy").run("fly deploy --remote-only").end_job()
```

## Config and secret routing

```python
from gheasy.core import GheasyConfig, gh_push_env

cfg = GheasyConfig(app='myapp', env_schema={
    'PORT': '5001',             # string default → gh variable set
    'DOMAIN': 'http://localhost:5001',
    'JWT_SCRT': None,           # None → gh secret set
    'RESEND_API_KEY': None,
    'CF_TUNNEL_TOKEN': None,
})
cfg.save()

import os
gh_push_env(dict(os.environ))   # routes each key by schema
```

Keys with `None` schema become GitHub secrets (`gh secret set`).
Keys with a string default become variables (`gh variable set`).
`gh_push_env` sends them all in one pass.

Push from a `.env` file without a schema:

```python
from gheasy.core import gh_secrets_from_file
gh_secrets_from_file('.env')
```

## Full project setup

```python
from gheasy.core import gh_setup

gh_setup('myapp', '1.2.3.4', 'myapp.com', deploy_cmd='./deploy.sh')
```

Creates `.gheasy/config.json`, writes `.github/workflows/gheasy.yml`,
and installs `uv run nbdev-prepare` as the pre-commit hook.

## New projects

```python
from gheasy.core import gh_new

gh_new('myorg/myrepo', template='nbdev')      # notebook-first library
gh_new('myorg/myrepo', template='fastship')   # plain Python package
```

The `fastship` template scaffolds the layout `ship-new` writes, then adds
`.github/workflows/release.yml` and sets `[tool.fastship].release = "tag"`,
so `ship-release` pushes `v<version>` and CI builds, publishes and writes the notes.
It skips the hatchling migration: fastship's layout is setuptools over a slugified
package directory, and migrating renames the wheel to the PyPI name.

Scaffold or wire up a package that already exists:

```python
from gheasy.core import gh_fastship_new, gh_fastship_release

gh_fastship_new('.', description='What it does')   # fastship's layout, keeps the .git already there
gh_fastship_release(path='.', test_cmd='pytest')   # the release workflow and the tag setting
```

## git LFS

```python
from gheasy import gh_lfs

gh_lfs(['*.mp3', '*.png', '*.jpg'], path='.')
```

Writes `.gitattributes` LFS tracking patterns and runs `git lfs install`.

## Repo health

```python
from gheasy.core import gh_check, gh_apply

findings = gh_check(path='.')
gh_apply(findings)
```

`gh_check` audits: pre-commit hook present, `.gitattributes` LFS block,
pyproject.toml on hatchling, dependabot config, repo topics.
`gh_apply` applies the fixes it can.

## GheasyConfig

```python
from gheasy import GheasyConfig

cfg = GheasyConfig.load('.')        # load from .gheasy/config.json
cfg.app                              # app name
cfg.env_schema                       # {key: default_or_None}
```

## Generated workflow

Written to `.github/workflows/gheasy.yml` with a
`# Managed by gheasy — do not edit directly` header.
Re-run `gh_workflow()` after editing the config to regenerate.

## Installing this skill into a repo

```python
from gheasy.core import mv_skill_md
mv_skill_md(dry_run=False)
```

Copies the bundled `SKILL.md` to `.claude/skills/gheasy/` and `.agents/skills/gheasy/`
under the repo root so coding agents pick it up. Defaults to `dry_run=True`.
