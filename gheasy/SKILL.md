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

The `uv_*_job` presets add the usual checkout, setup-uv, install,
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
    'PORT': '5001',             # string default -> gh variable set
    'DOMAIN': 'http://localhost:5001',
    'JWT_SCRT': None,           # None -> gh secret set
    'RESEND_API_KEY': None,
    'CF_TUNNEL_TOKEN': None,
})
cfg.save()

import os
gh_push_env(dict(os.environ))   # sends each key as a secret or variable, by schema
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

`gh_setup` creates `.gheasy/config.json`, writes `.github/workflows/gheasy.yml`,
and installs `uv run nbdev_prepare` as the pre-commit hook.

## git LFS

```python
from gheasy import gh_lfs

gh_lfs(['*.mp3', '*.png', '*.jpg'], path='.')
```

Runs `git lfs install`, then `git lfs track` for each pattern, which writes the patterns to `.gitattributes`. It also saves the patterns to the config.

## Repo health

```python
from gheasy.core import gh_check, gh_apply

findings = gh_check(path='.')
gh_apply(findings)
```

`gh_check` checks for a pre-commit hook, an LFS section in `.gitattributes`
between the comments `# BEGIN gheasy-lfs` and `# END gheasy-lfs`,
a `pyproject.toml` that uses hatchling, a dependabot config and repo topics.
The dependabot and topics checks need a GitHub token.
`gh_apply` runs the fixes that exist. The GitHub findings have no fix.

## GheasyConfig

```python
from gheasy import GheasyConfig

cfg = GheasyConfig.load('.')        # reads .gheasy/config.json
cfg.app                              # app name
cfg.env_schema                       # {key: default_or_None}
```

## Generated workflow

`gh_workflow` writes it to `.github/workflows/gheasy.yml` with a
`# Managed by gheasy — do not edit directly` header.
Edit the config, then run `gh_workflow()` again to regenerate the file.

## Installing this skill into a repo

```python
from gheasy.core import mv_skill_md
mv_skill_md(dry_run=False)
```

Copies the bundled `SKILL.md` to `.claude/skills/gheasy/` and `.agents/skills/gheasy/`
under the repo root, where coding agents read it. The default is `dry_run=True`.
