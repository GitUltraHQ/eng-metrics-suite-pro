# GitUltra Paid Tier — Quick Start

Welcome! You've been issued a **license key** for GitUltra's paid-tier
tools: `eng-api`, `rewrite-ratio`, and `gitultra-mcp`. This guide
covers everything you need to start running them.

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

## Running `gitultra-mcp`

An MCP ([Model Context Protocol](https://modelcontextprotocol.io/))
server so an AI agent -- Claude Desktop, Claude Code, or anything else
that speaks MCP -- can query the same metrics `eng-api` exposes,
conversationally. It's a thin layer on top of `eng-api`: needs a
running `eng-api` instance to actually do anything, and reuses that
same `API_KEY` for its own auth (one secret, not two).

**Already using this repo's `docker-compose.yml`?** `gitultra-mcp` is
already wired in -- just make sure your `.env`'s `GITULTRA_LICENSE_KEY`
covers it (see below) and `GITULTRA_MCP_PORT` is set if you want a
port other than the default `8100`.

**Running it standalone instead?**

```
docker run --rm -p 8100:8000 \
    -e ENG_API_BASE_URL=http://eng-api:8000 \
    -e ENG_API_KEY=<same value as eng-api's API_KEY> \
    -e GITULTRA_LICENSE_KEY=<your key> \
    ghcr.io/gitultrahq/gitultra-mcp:latest
```

Then connect a client. Claude Code:

```
claude mcp add --transport http gitultra http://localhost:8100/mcp \
    -H "Authorization: Bearer <your ENG_API_KEY value>"
```

Claude Desktop: add to your MCP server config --

```json
{
  "mcpServers": {
    "gitultra": {
      "type": "http",
      "url": "http://localhost:8100/mcp",
      "headers": {
        "Authorization": "Bearer <your ENG_API_KEY value>"
      }
    }
  }
}
```

Full tool list and details: see the
[gitultra-mcp README](https://github.com/GitUltraHQ/gitultra-mcp#readme).

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
