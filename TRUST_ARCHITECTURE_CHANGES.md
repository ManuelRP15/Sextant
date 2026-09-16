# Trust Architecture Changes

**Version 1 · 2026-09-16** (`DECISIONS.md #322`; the classification itself is `RELEASE_SECURITY.md` §4)

Sextant tells its users six things about itself, in its own Settings, on its website and in its store
listing. A change that would make any of those things less true is not an ordinary change, whatever
its size in lines. This page is the process for such a change: what counts, what it must update, what
detects it automatically and what only a person can do.

---

## 1. What counts

Any change that introduces, widens or alters one of these:

| Area | Examples |
|---|---|
| **Telemetry or analytics** | usage statistics, crash reporting, an error-reporting SDK, a "ping" of any kind |
| **A server or an account** | a Sextant backend, sign-in, cloud sync, a licence server, shared state |
| **AI or model APIs** | any provider called with org content, including under the user's own key |
| **Permissions** | a Chrome permission, an optional permission, a host permission, `externally_connectable` |
| **Network destinations** | a `connect-src` entry, a URL origin in the bundle, a new module that performs a request |
| **Remote resources** | a script, stylesheet, font, image or configuration loaded from anywhere but the package |
| **Authentication** | anything other than the org's own session cookie, read per request |
| **Data transmission** | any value that reaches a destination other than the org it came from |
| **Stored data** | a new category in `src/shared/data-inventory.ts`, a longer retention, a new storage mechanism |
| **The website or the demo** | a script on the site, a request from the demo, a cookie, a third-party resource |

A change that only *removes* one of these (a permission narrowed, a destination dropped) is still a Trust
Architecture Change: the disclosures must follow it, and a user is entitled to see it in the changelog.

## 2. What such a change must update, in the same change

1. `THREAT_MODEL.md` — the threat it introduces or removes, and the mitigation.
2. `src/shared/trust-facts.ts` — if a statement stops being true, it is withdrawn or reworded there first;
   the Settings panel, the website and the store copy follow from it (`security/trust-claims.test.ts`).
3. `src/shared/permission-explanations.ts` and `src/shared/data-inventory.ts` — a new permission or stored
   category must be explained where the code declares it; `DATA_HANDLING.md` and `PERMISSIONS.md` are then
   regenerated (`npm run docs:data-handling`).
4. `PRIVACY.md` and `PRIVACY.es.md` — a new version row in §12, in both languages.
5. `SECURITY.md` §4 and §5 — what is protected and what cannot be.
6. `CWS_PRIVACY_DISCLOSURES.md` — the store's answers.
7. `CHANGELOG.md` — under **Security** or **Privacy**, in plain words, including "permission added" or
   "new destination" when that is what happened.
8. `release-metadata/trust-baseline.json` — the approved surface, in its own reviewed commit.
9. The website — automatically, because it is built from 2, 3 and 4; nothing to edit by hand unless the
   words of `scripts/website/copy.ts` describe the old behaviour.

## 3. What detects it without anyone remembering

| Detection | Where | Fails |
|---|---|---|
| A permission, host, content-script match or CSP differs from the approved baseline | `src/security/trust-surface.test.ts` | `npm test` |
| A request primitive outside the one transport module | `src/security/network-boundary.test.ts` | `npm test` |
| A `chrome.storage` key the inventory does not classify | `src/security/data-inventory.test.ts` | `npm test` |
| A public document that repeats a statement differently, or uses a forbidden absolute | `src/security/trust-claims.test.ts`, `website.test.ts` | `npm test` |
| `DATA_HANDLING.md` or `PERMISSIONS.md` out of date | `src/security/data-handling-doc.test.ts` | `npm test` |
| A URL origin, analytics endpoint, AI provider marker, `eval`, WebSocket or beacon in the built package | `scripts/release/scan-dist.mjs` | every release build, `npm run release:verify` |
| A request from the demo, a script on the site, a cookie, a third-party resource | `scripts/demo/scan-demo-dist.mjs`, `demo-qa/`, `website.test.ts` | `npm run release:verify` |
| A change to any security-boundary file, bundled package or install script against the previous release | `scripts/release/release.mjs` (gate 13) | the release is classified **security-sensitive** and refuses to proceed without `--approve-trust-change "<why>"` |

The first release is security-sensitive by definition: every user is being asked to trust all of it.

## 4. What only a person does

- Reads the trust diff in full before approving it, and writes the reason into the release
  (`--approve-trust-change`), where `provenance.json` records it.
- Decides whether a new permission is worth the install-time warning it adds, and whether the feature
  could work with less.
- Updates the store listing's privacy practices from `CWS_PRIVACY_DISCLOSURES.md` — the form is not
  connected to the repository.
- For a server, an account, analytics or an AI capability: treats it as a product decision with a written
  privacy analysis before any code lands, because each of them changes what kind of product Sextant is —
  no server holding customer data is a deliberate, permanent property of the product, not a limitation to
  grow out of.

## 5. Pull requests

The pull-request template asks whether the change touches any area in §1. Answering *yes* means §2's list
is part of the change, not a follow-up.
