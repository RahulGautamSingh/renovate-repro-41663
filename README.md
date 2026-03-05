# Minimal Reproduction: renovatebot/renovate#41663

## Problem

When `registryUrls` (or `defaultRegistryUrls`) is set at the top level of `renovate.json`,
it propagates to **all** datasources — including `github-tags` and `github-releases`.

This causes the GitHub GraphQL fetcher to construct a GraphQL endpoint from a non-GitHub URL
(e.g. `https://pypi.org/pypi/api/graphql`) and fire a POST request that always returns HTTP 405.

## Reproduction

This repo has:
- `registryUrls: ["https://pypi.org/pypi/"]` set at the **top level** of `renovate.json`
- `requirements.txt` — triggers `pip` datasource (where PyPI URL is valid)
- `.github/workflows/ci.yml` — triggers `github-tags` datasource for Actions (where PyPI URL is **not** valid)

<details>

<summary> Debug Logs </summary>

```
DEBUG: Found 2 package file(s) (repository=RahulGautamSingh/renovate-repro-41663)
 INFO: Dependency extraction complete (repository=RahulGautamSingh/renovate-repro-41663, baseBranch=main)
       "stats": {
         "managers": {
           "github-actions": {"fileCount": 1, "depCount": 4},
           "pip_requirements": {"fileCount": 1, "depCount": 1}
         },
         "total": {"fileCount": 2, "depCount": 5}
       }
DEBUG: hostRules: no authentication for pypi.org (repository=RahulGautamSingh/renovate-repro-41663)
DEBUG: Using queue: host=pypi.org, concurrency=16 (repository=RahulGautamSingh/renovate-repro-41663)
DEBUG: POST https://pypi.org/pypi/api/graphql = (code=ERR_NON_2XX_3XX_RESPONSE, statusCode=405 retryCount=0, duration=392) (repository=RahulGautamSingh/renovate-repro-41663)
DEBUG: Datasource connection error (repository=RahulGautamSingh/renovate-repro-41663)
       "datasource": "github-tags",
       "packageName": "actions/setup-python",
       "url": "https://pypi.org/pypi/api/graphql/",
       "errCode": "ERR_NON_2XX_3XX_RESPONSE"
DEBUG: Failed to look up github-tags package actions/setup-python (repository=RahulGautamSingh/renovate-repro-41663, packageFile=.github/workflows/ci.yml, dependency=actions/setup-python)
DEBUG: POST https://pypi.org/pypi/api/graphql = (code=ERR_NON_2XX_3XX_RESPONSE, statusCode=405 retryCount=0, duration=1514) (repository=RahulGautamSingh/renovate-repro-41663)
DEBUG: Datasource connection error (repository=RahulGautamSingh/renovate-repro-41663)
       "datasource": "github-releases",
       "packageName": "actions/python-versions",
       "url": "https://pypi.org/pypi/api/graphql/",
       "errCode": "ERR_NON_2XX_3XX_RESPONSE"
DEBUG: Failed to look up github-releases package actions/python-versions (repository=RahulGautamSingh/renovate-repro-41663, packageFile=.github/workflows/ci.yml, dependency=actions/python-versions)
DEBUG: POST https://pypi.org/pypi/api/graphql = (code=ERR_NON_2XX_3XX_RESPONSE, statusCode=405 retryCount=0, duration=1296) (repository=RahulGautamSingh/renovate-repro-41663)
DEBUG: Datasource connection error (repository=RahulGautamSingh/renovate-repro-41663)
       "datasource": "github-tags",
       "packageName": "actions/checkout",
       "url": "https://pypi.org/pypi/api/graphql/",
       "errCode": "ERR_NON_2XX_3XX_RESPONSE"
DEBUG: Failed to look up github-tags package actions/checkout (repository=RahulGautamSingh/renovate-repro-41663, packageFile=.github/workflows/ci.yml, dependency=actions/checkout)

```

</details>

## Expected behavior

Renovate should emit a config warning telling the user that a non-GitHub `registryUrl`
is configured for a `github-tags`/`github-releases` datasource.

## Actual behavior

Renovate silently fires `POST https://pypi.org/pypi/api/graphql` → HTTP 405, with no
user-visible warning that the config is wrong.

Related: https://github.com/renovatebot/renovate/issues/41663


