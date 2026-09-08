# eng-metrics-suite-pro

New here? See [QUICKSTART.md](QUICKSTART.md) for the short version.

A ready-to-run `docker-compose.yml` for paying customers (Team and
Enterprise tiers -- they share the same software bundle, Enterprise's
difference is extra services, not extra features), bundling the same
free-tier pipeline as [eng-metrics-suite](https://github.com/GitUltraHQ/eng-metrics-suite)
alongside paid-tier add-ons, so a customer doesn't have to hand-edit
their own compose file to wire those in.

**This repo is not itself a security boundary.** The compose file
contains no proprietary logic -- just orchestration (image names + env
vars). The actual protection is the **signed license key**
(`GITULTRA_LICENSE_KEY`) each paid add-on (`eng-api`, and any future
ones) verifies at startup -- their GHCR images are public. This repo
being private is about not casually publicizing the paid bundle's shape
to anyone browsing the public repo, not a second gate. See each add-on's
own repo for its specific distribution details, or [QUICKSTART.md](QUICKSTART.md)
for the short version.

## What's included

Everything in `eng-metrics-suite` (`postgres`, `git-processor`,
`pr-processor`, `eng-reports` -- including its second script,
`allocation_report.py`, for investment allocation -- and optional
`issue-processor` for change-failure-rate/MTTR and/or investment
allocation, each independently configurable), plus:

- **`eng-api`** -- REST/JSON API for dashboard integrations. Long-running
  service, exposed on `ENG_API_PORT` (default 8000). See
  [eng-api](https://github.com/GitUltraHQ/eng-api)'s own README for the
  full endpoint list, auth, and config -- including
  `/v1/change-failure-rate`/`/v1/mttr`, which return real data only once
  `issue-processor` is configured (see `.env.example`'s `JIRA_*` vars).
- **`gitultra-mcp`** -- MCP server so an AI agent (Claude Desktop, Claude
  Code, etc.) can query the metrics above conversationally, by wrapping
  `eng-api`'s endpoints as tools. Long-running, exposed on
  `GITULTRA_MCP_PORT` (default 8100). Public GHCR image, license-gated
  -- see [gitultra-mcp](https://github.com/GitUltraHQ/gitultra-mcp)'s own
  README for setup and client-connection details.

**Not included in the compose file:** `rewrite-ratio`. It's a one-shot
diagnostic CLI, not a long-running service, so it doesn't belong in a
`docker compose up` stack -- run it directly per its own README:

```
docker run --rm -v /path/to/repo:/repo:ro ghcr.io/gitultrahq/rewrite-ratio:latest \
    --repo /repo --author alice@example.com --start 2026-01-01 --end 2026-08-24
```

## Quickstart

```
cp .env.example .env
# edit .env: set POSTGRES_PASSWORD, ENG_API_KEY, GITULTRA_LICENSE_KEY,
# and whichever vendor token(s) you're importing from

sudo mkdir -p /var/lib/eng-metrics-suite
sudo chown "$(id -u):$(id -g)" /var/lib/eng-metrics-suite
mkdir -p reports

docker compose up -d
```

Same setup as `eng-metrics-suite` otherwise -- see
[eng-metrics-docs](https://docs.gitultra.com/) for
Getting Started / Discovering Repos / Running Workers / Generating
Reports, none of which differ here. This repo's README only covers
what's different: the paid add-ons. `docker-compose.yml` also declares a
`git_mirrors` volume for `git-processor`'s persistent per-repo clone
cache -- Docker manages that one itself, no `mkdir`/`chown` needed.

## Requirements

- Docker + Docker Compose
- A signed license key for `eng-api` (issued to you after purchase --
  contact support@gitultra.com). No `docker login`/package grant needed:
  `ghcr.io/gitultrahq/eng-api` is a **public** image, gated instead by
  `GITULTRA_LICENSE_KEY` -- see `eng-api`'s own README's "Distribution"
  section for details.

## Distribution (this repo's own access)

You were added as a **Collaborator** (Read access) on this repo
directly, for convenience -- see the note above for why this repo isn't
itself a security boundary. Unlike the manual GHCR package-grant model
this repo used to describe, `eng-api`'s actual access control is now the
signed `GITULTRA_LICENSE_KEY` (see "Requirements" above); removing this
repo's Collaborator access doesn't revoke that key.

## License

See [LICENSE](LICENSE) for this bundle. Each individual paid component
(`eng-api`, `rewrite-ratio`, etc.) is separately licensed under its own
repo's terms.
