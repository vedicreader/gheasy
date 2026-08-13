# gheasy

gheasy makes GitHub Actions, secrets, and deploys operable without hand-writing YAML.
Config lives in `.gheasy/config.json`; the managed workflow is `.github/workflows/gheasy.yml`.

## Happy path

```bash
# Library package
gheasy setup mylib --workflow-preset library

# App (FastHTML / VPS)
gheasy setup myapp 1.2.3.4 myapp.com --workflow-preset fasthtml --deploy-cmd ./deploy.sh
gheasy secrets
gheasy gh-deploy-key-setup deploy_key

gheasy status
gheasy logs
```

Python equivalent:

```python
from gheasy import gh_setup, gh_secrets, gh_deploy_key_setup, gh_status, gh_logs

gh_setup('myapp', '1.2.3.4', 'myapp.com',
         workflow_preset='fasthtml', deploy_cmd='./deploy.sh')
gh_secrets('.env')
gh_deploy_key_setup('deploy_key')
gh_status(); gh_logs()
```

## Presets

| Preset | Jobs |
|---|---|
| `python` | test + lint (+ PR) |
| `library` | test + lint + PyPI on release (+ PR) |
| `fasthtml` | test (+ PR); SSH deploy if host + `deploy_cmd` |
| `nodejs` / `rust` / `go` | language CI (+ PR) |

```bash
gheasy enable test lint publish_pypi
gheasy workflow
```

## Secrets

```python
gh_secrets('.env')                 # all keys → secrets (default)
gh_secrets('.env', schema=True)    # cfg.env_schema: None→secret, str→variable
```

Requires authenticated `gh` CLI.

## Logs & status

```bash
gheasy runs
gheasy logs          # latest gheasy.yml run
gheasy watch
gheasy status        # local deploy records + recent runs
```

## Custom DSL (optional)

```python
from gheasy.workflow import Workflow

wfb = Workflow("ci")
wfb.on.push(branches=["main"]).pull_request()
wfb.uv_lint_job()
wfb.uv_test_job(needs="lint")
wfb.uv_pypi_job(needs="test")
wfb.build().save(".github/workflows/ci.yml")
```

## Repo health / LFS

```python
from gheasy import gh_lfs, gh_check, gh_apply
gh_lfs(['*.mp3', '*.png', '*.jpg'])
gh_apply(gh_check(path='.'))
```

## Installing this skill into a repo

```python
from gheasy.core import mv_skill_md
mv_skill_md(dry_run=False)
```
