# GitUltra Paid Tier — Quick Start

Welcome! You've been issued a **license key** for GitUltra's paid-tier
tools: `eng-api` and `rewrite-ratio`. This guide covers everything you
need to start running them.

## Your license key

You should have received a long string (starts with `eyJ...`) alongside
this guide — that's your license key. Treat it like a password: it's
tied to you and valid for **90 days**. When it's close to expiring, just
ask us for a renewal (support@gitultra.com) — it's a fresh key issued
the same way, no separate "renew" process.

Both tools read the key from a single environment variable:

```
GITULTRA_LICENSE_KEY=<the key you were issued>
```

No `docker login` or GitHub account needed — both images are public on
GHCR. The license key is the only thing gating access.

## Running `rewrite-ratio`

One-shot CLI, no setup beyond Docker. Bind-mount the repo you want to
analyze read-only:

```
docker run --rm -v /path/to/repo:/repo:ro \
    -e GITULTRA_LICENSE_KEY=<your key> \
    ghcr.io/gitultrahq/rewrite-ratio:latest \
    --repo /repo --author alice@example.com --start 2026-01-01 --end 2026-08-24
```

Full options and how the ratio is calculated: see the
[rewrite-ratio README](https://github.com/GitUltraHQ/rewrite-ratio#readme).

## Running `eng-api`

Long-running REST API over the same metrics database `git-processor`/
`pr-processor` populate.

**Already using this repo's `docker-compose.yml`?** Just set
`GITULTRA_LICENSE_KEY` in your `.env` (see this repo's own
[Quickstart](README.md#quickstart)) — `eng-api` is already wired in.

**Running it standalone instead?**

```
docker run --rm -p 8000:8000 \
    -e DATABASE_URL=postgres://user:pass@host/db \
    -e API_KEY=<a secret you choose> \
    -e GITULTRA_LICENSE_KEY=<your key> \
    ghcr.io/gitultrahq/eng-api:latest
```

Endpoints, auth details, and optional config: see the
[eng-api README](https://github.com/GitUltraHQ/eng-api#readme).

## What the license key does (and doesn't) protect

Straight talk: the key stops casual, zero-effort use of the now-public
images with no credential at all, and forging a valid key without our
private signing key is cryptographically infeasible. It does **not**
stop someone from sharing a valid key, or a technically inclined user
from patching the (open-source, unobfuscated) check out of their own
copy of the image. Please don't share your key — it's tied to you, and
the honor system is a real part of how this works at our current scale.

## Questions / issues

support@gitultra.com — license renewals, access issues, or anything
else.
