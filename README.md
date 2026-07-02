# last30days (light)

**Research what real people actually say about any topic in the last 30 days.**
An AI agent searches Reddit, X, YouTube, TikTok, Instagram, Hacker News, Polymarket,
GitHub, and the web in parallel, scores results by real engagement (upvotes, likes,
money, not editors), and synthesizes one cited brief.

> This is a slimmed **"light" fork** of
> [mvanhorn/last30days-skill](https://github.com/mvanhorn/last30days-skill) that keeps
> **only the Claude skill itself**. The multi-platform plugin packaging (Codex, Gemini,
> the Claude Code marketplace bundle), the standalone Go MCP server, demo assets, the
> test suite, and dev tooling have been removed. The runtime spec in
> [skills/last30days/SKILL.md](skills/last30days/SKILL.md) is the source of truth.

## What's in this fork

```
skills/last30days/
  SKILL.md              # the skill spec (source of truth for commands and setup)
  scripts/
    last30days.py       # the research engine
    store.py            # optional SQLite persistence (--store)
    watchlist.py        # scheduled runs + optional Slack/webhook delivery
    briefing.py         # daily / weekly digests
    setup-keychain.sh   # optional: store keys in the macOS Keychain
    setup-pass.sh       # optional: store keys in pass(1)
    lib/                # engine internals: per-source adapters, vendored X client
  references/
LICENSE                 # MIT (from the original project)
```

The skill is self-contained: it has **zero pip dependencies** (the X client is vendored
under `lib/vendor/`) and every entrypoint imports only from the in-tree `lib/`.

## Install (Claude Code)

Because this is the bare skill, install it by pointing Claude at the skill directory:

```bash
git clone https://github.com/YogirajA/last30days-skill-light.git
ln -s "$(pwd)/last30days-skill-light/skills/last30days" ~/.claude/skills/last30days
```

Then run `/last30days <topic>` in Claude Code. Requires **Python 3.12+**; Node.js is used
for X search and `yt-dlp` for YouTube transcripts (both optional).

Preview what a run will do before the first one (reads no cookies, writes no files,
makes no network calls):

```bash
python3 skills/last30days/scripts/last30days.py --preflight
```

Reddit (with comments), Hacker News, Polymarket, and GitHub work immediately with zero
configuration. Run it once and the setup wizard walks you through unlocking the rest.

## Example

```
/last30days Peter Steinberger
```

You have a meeting tomorrow, so you Google them and get their 2023 LinkedIn. This gives
you what they are actually doing this month: who they joined, the PRs they are shipping
and at what merge rate, what they are building, and the Reddit thread debating whether
they are a hero or "insufferable." Scattered across X posts, Reddit threads, YouTube
transcripts, and GitHub commits, none of it on Google.

## Sources, scored by the people

| Source | What it gives you |
|--------|-------------------|
| **Reddit** | Top comments with upvote counts, free via public JSON. No key required. |
| **X / Twitter** | Hot takes and expert threads. Uses your browser session or a provider key. |
| **YouTube** | Full transcripts, searched for the few sentences that matter. |
| **TikTok / Instagram / Threads / Pinterest** | Creator reach and engagement (ScrapeCreators key). |
| **Hacker News** | Developer consensus, points and comment counts. |
| **Polymarket** | Not opinions, odds backed by real money. |
| **GitHub** | PR velocity, top repos, release notes (person-mode and topic-mode). |
| **Bluesky** | AT Protocol posts (free app password). |
| **Perplexity** | Grounded Sonar synthesis and Deep Research (key-gated, opt-in). |
| **Web** | Editorial coverage as one signal among many. |

Results are ranked by what real people engaged with (social relevancy, not SEO), and
duplicate stories across sources are merged into one cluster.

## Bring your own keys

Every key is optional. The engine degrades to whatever is available.

| Sources | What you need | Cost |
|---------|---------------|------|
| Reddit (with comments) + HN + Polymarket + GitHub | Nothing | Free |
| X / Twitter | Log into x.com in a browser, or set `XQUIK_API_KEY` / `XAI_API_KEY` | Cookies free; keys provider-specific |
| YouTube | `yt-dlp` on PATH | Free |
| Bluesky | App password from bsky.app | Free |
| TikTok + Instagram + Threads + Pinterest + YouTube comments | `SCRAPECREATORS_API_KEY` | 10k free calls, then pay as you go |
| Perplexity | `PERPLEXITY_API_KEY`, or `OPENROUTER_API_KEY` as Sonar fallback | Pay as you go |
| Web search | `BRAVE_API_KEY` | 2,000 free queries/month |

Set keys in your shell, in `~/.config/last30days/.env` (written `0600`), or on macOS in
the Keychain via `skills/last30days/scripts/setup-keychain.sh`. Keys are read from the
environment and keychain only; they are never hardcoded or logged.

## Configuration essentials

- **Where files are saved:** `LAST30DAYS_MEMORY_DIR` defaults to `~/Documents/Last30Days/`.
  Override per run with `--save-dir <path>`, or write an exact file with `--output <file>`.
- **Trend monitoring:** add `--store` to persist runs into SQLite, then use
  [`scripts/watchlist.py`](skills/last30days/scripts/watchlist.py) for scheduled runs
  (with optional Slack / webhook delivery on new findings) and
  [`scripts/briefing.py`](skills/last30days/scripts/briefing.py) for digests.
- **Output formats:** `--emit compact|json|context|md|html|brief`. The `html` and `brief`
  formats save a self-contained, dark-mode, offline-friendly file.

See [skills/last30days/SKILL.md](skills/last30days/SKILL.md) for the full command surface.

## How it works

1. **You type a topic** (person, company, product, "X vs Y").
2. **The agent resolves who matters:** the right X handles, GitHub repos, subreddits,
   hashtags, and channels, before any search fires.
3. **All sources are searched in parallel,** scored by engagement, relevance, and freshness.
4. **Same story across sources is merged** into one cluster.
5. **Everything is synthesized into one cited brief,** ranked by what people engaged with.

## Security and privacy

This fork was reviewed before publishing. Notes worth knowing:

- **Browser-cookie use is opt-in and first-party only.** When enabled, the skill reads
  your X `auth_token`/`ct0` from your browser to search x.com. Reads are domain-scoped,
  default-off, consent-gated, never persisted, and sent **only** to `x.com`. Disable
  entirely with `--no-browser-cookies`.
- **Third-party query egress:** enabling sources like ScrapeCreators, Xquik, Jina, or
  Arctic-Shift sends your query text (and that source's own key) to those services. Each
  receives only its own key, never your X cookies.
- **Publishing:** `--emit=html` saves a local file. There is no telemetry, analytics, or
  phone-home in the skill.

## Credits and license

Original project by **Matt Van Horn** ([@mvanhorn](https://github.com/mvanhorn)), MIT
license. v3 engine architecture by [@j-sperling](https://github.com/j-sperling). This
light fork keeps only the Claude skill and does not add or change functionality; see the
upstream repo for the full multi-platform distribution, changelog, and contributors.
