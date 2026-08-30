# eng-metrics-suite-pro

A ready-to-run `docker-compose.yml` for paying customers (Team and
Enterprise tiers -- they share the same software bundle, Enterprise's
difference is extra services, not extra features), bundling the same
free-tier pipeline as [eng-metrics-suite](https://github.com/jsooter/eng-metrics-suite)
alongside paid-tier add-ons, so a customer doesn't have to hand-edit
their own compose file to wire those in.

**This repo is not itself a security boundary.** The compose file
contains no proprietary logic -- just orchestration (image names + env
vars). The actual protection is the **private GHCR packages** it
references (`eng-api`, and any future add-ons); this repo being private
is about not casually publicizing the paid bundle's shape to anyone
browsing the public repo, not a second gate. See each add-on's own repo
for its specific distribution/access-grant process.

## What's included

Everything in `eng-metrics-suite` (`postgres`, `git-processor`,
`pr-processor`, `eng-reports`), plus:

- **`eng-api`** -- REST/JSON API for dashboard integrations. Long-running
  service, exposed on `ENG_API_PORT` (default 8000). See
  [eng-api](https://github.com/jsooter/eng-api)'s own README for the
  full endpoint list, auth, and config.

**Not included in the compose file:** `rewrite-ratio`. It's a one-shot
diagnostic CLI, not a long-running service, so it doesn't belong in a
`docker compose up` stack -- run it directly per its own README:

```
docker run --rm -v /path/to/repo:/repo:ro ghcr.io/jsooter/rewrite-ratio:latest \
    --repo /repo --author alice@example.com --start 2026-01-01 --end 2026-08-24
```

## Quickstart

```
cp .env.example .env
# edit .env: set POSTGRES_PASSWORD, ENG_API_KEY, and whichever vendor
# token(s) you're importing from

sudo mkdir -p /var/lib/eng-metrics-suite
sudo chown "$(id -u):$(id -g)" /var/lib/eng-metrics-suite
mkdir -p reports

docker compose up -d
```

Same setup as `eng-metrics-suite` otherwise -- see
[eng-metrics-docs](https://jsooter.github.io/eng-metrics-docs/) for
Getting Started / Discovering Repos / Running Workers / Generating
Reports, none of which differ here. This repo's README only covers
what's different: the paid add-ons.

## Requirements

- Docker + Docker Compose
- Pull access to the private `ghcr.io/jsooter/eng-api` package (granted
  to you individually after purchase -- see `eng-api`'s README if you
  need to re-authenticate `docker login ghcr.io`)

## Distribution (this repo's own access)

Same manual, per-customer pattern as the packages it references: you
were added as a **Collaborator** (Read access) on this repo directly.
This repo's contents aren't sensitive on their own (see the note above),
so this is about convenience, not a second security layer -- don't
assume removing repo access alone revokes anything meaningful without
also reviewing package-level grants.

## License

See [LICENSE](LICENSE) for this bundle. Each individual paid component
(`eng-api`, `rewrite-ratio`, etc.) is separately licensed under its own
repo's terms.
