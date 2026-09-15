# RapTech Releases

This repository is how a RapTech computer finds out that a newer version exists.

It holds **no source code**. What is here is one small file per product, plus a
GitHub release per build with the executables attached.

RapTech itself is two separate programs, and they update independently:

| Product | What it is | Manifest | Release tags |
| --- | --- | --- | --- |
| `client` | the terminal — the coin-operated timer on a shop PC | [`client/latest.json`](client/latest.json) | `client-vX.Y.Z` |
| `server` | the counter — the staff dashboard behind the desk | [`server/latest.json`](server/latest.json) | `server-vX.Y.Z` |

A counter on 1.4.1 with a floor of terminals on 1.2.0 is a perfectly normal
shop. Neither waits for the other.

## Why this repository is public

A shop PC has to be able to ask "is there a newer version?" without credentials.
The RapTech source is private; this is not, and it contains nothing that needs to
be.

The programs read the raw manifest file directly —
`raw.githubusercontent.com/.../main/<product>/latest.json` — and deliberately not
the GitHub API. No token, no rate limit, and no JSON shape that can change under
a build that shipped two years ago.

## What a manifest says

```json
{
  "product": "client",
  "version": "1.2.0",
  "released": "2026-09-16",
  "notes": "What changed, in one line.",
  "url": "https://github.com/.../releases/download/client-v1.2.0/RapTech.exe",
  "size": 14680064,
  "sha256": "…"
}
```

A client manifest may also carry `helperUrl`, `helperSize` and `helperSha256`
for `systemRunTime.exe`, the watchdog that restarts the terminal. A terminal
ships as two programs and they are installed together — a shop with one and not
the other is the state those fields exist to prevent. They are optional so that
the counter, which has no watchdog, goes on reading a file written for both.

**`"version": "0.0.0"` with an empty `url` means nothing has been published for
that product yet.** Every RapTech build reads that as "you are up to date", not
as a failure — a shop must never be shown an error because a release has not
been cut. Both manifests start out that way.

## What a RapTech computer checks before installing anything

The download is not trusted because it came from here. It is trusted because:

- the URL is **HTTPS and a GitHub host**, checked on the first request *and on
  every redirect* — a redirect being exactly where an open-ended download would
  be steered;
- the **size and SHA-256 match the manifest**, or the file is discarded. A
  captive portal's login page fails the hash instead of being installed;
- for a terminal, **both** files are downloaded and hash-checked *before* either
  one is swapped in.

## Publishing

Releases are not cut by hand and not by CI. They are published from the private
source repository with `scripts/publish-release.sh`, which uploads the release
and its assets first and rewrites the manifest last — so an upload that fails
halfway leaves every shop on the version it already has.

## Reporting a problem

Issues here are welcome for anything about a *download*: a release asset that is
missing, a manifest that looks wrong, a version that will not install. Bugs in
RapTech itself are better raised with whoever supplied your shop, since the
source is not here.
