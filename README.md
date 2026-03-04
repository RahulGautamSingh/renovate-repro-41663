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

## Expected behavior

Renovate should emit a config warning telling the user that a non-GitHub `registryUrl`
is configured for a `github-tags`/`github-releases` datasource.

## Actual behavior

Renovate silently fires `POST https://pypi.org/pypi/api/graphql` → HTTP 405, with no
user-visible warning that the config is wrong.

Related: https://github.com/renovatebot/renovate/issues/41663
