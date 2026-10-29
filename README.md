# Checkout without persisted credentials

[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/infralicious/actions-checkout/badge)](https://securityscorecards.dev/viewer/?uri=github.com/infralicious/actions-checkout)
[![Test](https://github.com/infralicious/actions-checkout/actions/workflows/test.yml/badge.svg)](https://github.com/infralicious/actions-checkout/actions/workflows/test.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

This action passes through to `actions/checkout`.
It uses the full upstream SHA, not a moving tag.
It keeps every upstream input and default.
Only `persist-credentials` changes.
It defaults to `false`, not `true`.

## Why this repo exists

Upstream keeps true as default for push convenience.
Upstream has not changed that default.
[Issue 485](https://github.com/actions/checkout/issues/485) asked for a change in 2021.
Upstream still uses true today.
This repo wraps upstream.
It does not copy upstream code.
It gives users a secure default without a fork.

## What problem it solves

Many workflows use checkout in many places.
One missed setting leaks a token.
Later code in the same job can then use that token.
That code can include untrusted code from a fork.
Upstream default stores a token for later git use.
[Issue 485](https://github.com/actions/checkout/issues/485) calls this a severe security issue.
[GitHub Security Lab](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/) links this default to pwn requests.
[ArtiPACKED research](https://dev.to/rxkov/github-actions-when-the-cicd-pipeline-becomes-a-secrets-exfiltration-vector-pea) found tokens in uploaded artifacts in 2024.
Older versions stored the token in `.git/config` directly.
Newer versions store it in a temp file instead.
Later steps can still use that temp file for git.
Current version keeps that temp file design.
False removes that later use by default.
This repo removes that miss by default.
You do not need to repeat the setting.
False does not fix all token risks.

## Switch from upstream

Use this diff to switch from upstream.
You remove the with block in both forms.
Diff shows step lines, not a patch file.

Tag form:

```diff
- uses: actions/checkout@<version>
-   with:
-     persist-credentials: false
+ uses: infralicious/actions-checkout@<version>
```

SHA form:

```diff
- uses: actions/checkout@<upstream-sha> # <version>
-   with:
-     persist-credentials: false
+ uses: infralicious/actions-checkout@<repo-sha> # <version>
```

Tag in this repo pins upstream SHA internally.
Use the commit SHA for the tag for full SHA pinning.
Find that SHA on the release page.
Use commit SHA, not tag object SHA.
Repo SHA is the tag commit, not upstream SHA.

## Use

```yaml
name: Test
on:
  push:
jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: infralicious/actions-checkout@<version>
      - run: ls
```

Replace <version> with the latest release tag.
Add a with block to the checkout step to pass an input.
Use fetch-depth: 0 in that block for full history.

You may omit `repository` and `token`.
The action then uses the workflow repository and token.
Composite defaults cannot evaluate workflow values directly.
The step adds that fallback at run time.
You may set `persist-credentials` to `true` when you need it.
With `false`, later steps cannot push with the stored token.
Configure git credentials yourself for a later push.

## Outputs

It returns `ref` from upstream.
It returns `commit` from upstream.

## Tags

Use exact version tag only, not floating major tag.
A floating major tag does not exist in this repo.
Floating tag cannot move after an immutable release.
Immutable release locks its tag to one commit.
Exact tag only keeps releases immutable.

## Inputs

Inputs match upstream exactly, except one change.
`persist-credentials` defaults to `false`.
See `action.yml` for the full input list.

## Limits

This action does not remove tokens from other steps.
It does not scan artifacts for tokens.
Use least privilege permissions in each job.
This repo does not support upstream versions older than the current tag.

## Gotchas when you switch to false

False does not break checkout itself.
Checkout step authenticates itself.
Initial submodules and LFS fetch still work.

Later git write fails.
Later git push fails.
Later git fetch fails.
Later git pull fails.
Error is 403 or authentication failed.
Error does not name persist-credentials.
Check local git config for extraheader or includeIf to diagnose.

Later private clones fail.
Later Go, npm, or pip git dependencies from private repos fail.
Later submodule update fails.
Later LFS pull fails.
Configure credentials for that specific step only.

Release tools fail with false.
Semantic release fails.
Release please fails.
Changelog push fails.
Tag push fails.
Badge push fails.
Put git write in a separate job with persist true.
Keep untrusted code out of that job.

SSH key also does not persist.
Checkout step uses SSH command env only during checkout.
Later git over SSH fails.

gh CLI still works with false.
gh uses a token directly, not git config.
Use gh for API work, not for git push.

GITHUB_TOKEN push does not trigger workflows.
Push with GITHUB_TOKEN will not start on push workflows.
Use a GitHub App token if downstream CI must run.
App token needs contents write.

False does not hide the token from job steps.
Token power comes from the permissions block.
Set contents read where no write is needed.
False is defense in depth, not a replacement for least privilege.

Manual credentials leak again if you restore them.
If you restore credentials by hand for a push, clean them up after push.
Do not upload artifacts that contain git config.
Do push as the last step, or in an isolated job.

gh auth setup-git is a workaround for a push job.
Run gh auth setup-git before push.
It configures git to use the gh token.
Use it only in the push job.

A second checkout with false does not fix push.
A second checkout with true does fix push.
True then persists the token for the rest of that job.

Commit creation still needs user config.
False does not set git user name or email.
Set user name and email before commit, regardless of persist.

Fork code in pull_request_target needs care.
Untrusted fork code cannot use a stored token with false.
That limit is intended.
If a job must push to a fork, configure credentials explicitly.
Do not run untrusted code in that same job.


See [CONTRIBUTING](CONTRIBUTING.md) for maintainer steps.
<!-- test required check, do not merge -->
