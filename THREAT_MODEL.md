# Sextant threat model

**Version 1 · 2026-09-15** (`DECISIONS.md #320`, `ROADMAP.md` `P46-TM1` … `P46-TM5`)

What Sextant protects, from whom, how, how each protection is known to work, and what remains. Every
mitigation names the file or test that holds it; a mitigation without one is marked as a process or an
open item, never presented as a control. The findings of the audit that produced this document are in
`AUDIT_2026-09-15.md`.

---

## 1. What is worth attacking

| Asset | Why it matters |
|---|---|
| **The Salesforce session** (`sid` cookie) | A bearer credential for the user's org: whoever holds it acts as that user until it expires. The most valuable thing Sextant touches. |
| **The ability to write to an org** | Sextant can change translations and deploy metadata as the user. Used against the user's intent, it changes a production org. |
| **Org content in local storage** | Metadata, translations, names of users, deployment records, history — per org, in `chrome.storage.local`. |
| **The update channel** | Whoever can publish a Sextant version runs code with Sextant's permissions on every installed browser. |
| **The release inputs** | Source, lockfile, dependencies, the build machine, the store account. |
| **The user's trust in what Sextant says about itself** | A false privacy statement is a harm even when no data is lost. |

## 2. Trust boundaries

```mermaid
flowchart LR
  subgraph Browser["Browser profile"]
    Page["Salesforce Lightning page<br/>(untrusted DOM and script)"]
    CS["Content script<br/>(isolated world, closed shadow root)"]
    SW["Service worker<br/>(cookies, storage, network)"]
    EP["Extension pages<br/>(Workspace, toolbar menu)"]
    Store[("chrome.storage.local")]
  end
  SF["Salesforce org API<br/>*.my.salesforce.com"]
  CWS["Chrome Web Store"]
  Dev["Developer machine<br/>and release pipeline"]

  Page -- DOM, events --> CS
  CS -- "runtime messages<br/>(allow-listed types)" --> SW
  EP -- runtime messages --> SW
  SW -- "HTTPS, bearer session<br/>(CSP: salesforce.com only)" --> SF
  SW --- Store
  EP --- Store
  Dev -- signed upload --> CWS
  CWS -- updates --> Browser
```

The boundaries that matter most:

1. **Page → content script.** Everything a Salesforce page renders, including text an attacker may control
   (a record name, a label value, a component's markup), reaches the content script.
2. **Content script → service worker.** The content script is the one privileged component exposed to
   page-controlled input; the worker holds the session, the storage and the network.
3. **Service worker → the network.** The only place data leaves the browser.
4. **Developer → store → every user.** An update runs with Sextant's permissions everywhere.

## 3. Adversaries

| Id | Adversary | Can | Cannot (by assumption) |
|---|---|---|---|
| A1 | **Hostile content in a Salesforce page** — a user or integration that controls record data, labels, component markup or a custom component's script | Put arbitrary text and markup in front of the content script; dispatch DOM events; run script in the page's own world | Read the content script's isolated world or a closed shadow root; call extension APIs |
| A2 | **A hostile website** (not Salesforce) | Everything any web page can do | Message Sextant (no `externally_connectable`); get the content script injected (Lightning only) |
| A3 | **Another installed extension** | Its own permissions | Reach Sextant's handlers: Sextant registers no external message listener |
| A4 | **A hostile file** the user imports | Arbitrary JSON of any size | — |
| A5 | **A compromised dependency** | Code at install time (scripts) or in the bundle | Pass the release gates unnoticed if it adds a destination, telemetry or an install script (§4, T8) |
| A6 | **A compromised publisher account or build machine** | Publish any package to every user | — this is the one the product cannot defend against by itself (T9) |
| A7 | **Someone with the user's browser profile or computer** | Read local storage and the live session | — out of scope for technical controls (T14) |
| A8 | **The user, making a mistake** | Deploy to the wrong org or to production | — the design helps here (T13) |

Out of scope: vulnerabilities in Salesforce, in the browser, or in the operating system; network attackers
against TLS; Salesforce administrators acting within their own org.

## 4. Threats, mitigations, evidence, residual risk

Evidence key: **E** enforced by the browser or platform · **T** automated test · **B** release build check ·
**P** process (a human step) · **O** open.

### T1 — The session is sent somewhere other than the user's org

- **Mitigations.** CSP `connect-src 'self' https://*.salesforce.com` on every production build
  (`manifest.config.ts`) — **E**, verified in a real browser against the built extension, with a positive
  control (`security-qa/csp-enforcement.pw.ts`). One transport performs every request and validates the
  destination with a real URL parser, refusing userinfo, ports, lookalike suffixes and non-HTTPS
  (`src/shared/salesforce-transport.ts`, `salesforce-origin.ts`; `src/shared/salesforce-origin.test.ts`) — **T**.
  No other module calls a request primitive (`src/security/network-boundary.test.ts`) — **T**. The release
  scan fails on any unapproved URL origin in the bundle (`scripts/release/scan-dist.mjs`) — **B**.
- **Residual.** The CSP allows any host under `salesforce.com`, not only the user's org — and that includes
  hosts anyone can register (a free Developer Edition org) and Salesforce's own unauthenticated intake
  hosts; the per-org rule is enforced by code and tests, not by the browser. The CSP governs *connections*
  only: opening a tab, a DNS hint (`dns-prefetch`, `preconnect`) or a meta refresh could carry data
  out without a `fetch`. Nothing in the product uses those primitives outside `safe-navigation.ts`
  (`src/security/navigation-boundary.test.ts`) — **T** — and the release scan fails the package on any
  of them the product never uses and reports where it opens tabs (`scripts/release/scan-dist.mjs`) —
  **B**. Both hold for the product's own code; against a compromised dependency (T8) or publisher (T9)
  they detect rather than prevent.

### T2 — The session leaks through logs, storage, messages or exports

- **Mitigations.** Read per request, returned to the caller, never written (`getSessionId` in
  `src/background/index.ts`). Static and runtime checks that the value never reaches a console line, the
  storage bucket, a message reply or a request to another host (`src/security/session-boundary.test.ts`) —
  **T**. Diagnostics are opt-in and redact credential shapes (`src/shared/diagnostics.ts`,
  `secret-redaction.ts`); exports redact and withhold credentials (`local-data.ts`) — **T**. The release
  scan fails on a session-shaped string in the bundle — **B**.
- **Residual.** A Salesforce error body echoed into a console warning is not redacted in the live console
  (it is not persisted).

### T3 — Page-controlled input makes the worker act for someone else (confused deputy)

- **Mitigations.** Every message is classified by sender before dispatch
  (`src/shared/message-guard.ts`): content scripts are limited to ten message types the in-page UI needs,
  must come from a Lightning origin, and must carry origins that match their sender where they name one;
  sizes are bounded. Deletion, settings, org switching and every other type are extension-page only.
  Refusals are silent to the sender and counted (`src/security/message-boundary.test.ts`, including a
  hostile content-script context against the real worker) — **T**.
- **Residual.** A content script context taken over by A1 (its *isolated world*, which page script cannot
  reach; a browser or extension defect would be needed) can still ask for what the tooltip can do —
  including queuing an edit and saving a translation. The production and unidentified-org confirmation
  (T13) is asked *by the content script itself*, so it does not bind a context that has been taken over;
  and Chrome grants such a context the same `chrome.storage` access the worker has, so the message
  policy bounds what the worker will *do*, not what the store holds. Page script in its own world gets
  none of this: the hotkey handlers ignore synthetic keyboard events (`isTrusted`, `#323`), and the
  tooltip's controls live inside a closed shadow root it cannot target.

### T4 — Org content becomes script in an extension page or an exported report

- **Mitigations.** React rendering only; no `dangerouslySetInnerHTML`, `innerHTML` assignment, `eval`,
  `Function` or string timers (`src/security/code-boundary.test.ts`) — **T**. Extension pages run under the
  CSP (`script-src 'self'`, no inline script) — **E**. The exported HTML report escapes every
  interpolated value and carries its own restrictive CSP (`src/shared/conformance-export.ts`) — **T**.
- **Residual.** None known.

### T5 — A URL built from org data navigates somewhere dangerous

- **Mitigations.** Every navigation sink goes through `src/shared/safe-navigation.ts`, which accepts only
  HTTPS Salesforce URLs or the extension's own pages; `javascript:`, `data:` and lookalike hosts are refused
  (`src/security/navigation-boundary.test.ts`) — **T**.
- **Residual.** None known.

### T6 — The page interferes with Sextant's UI

- **Mitigations.** The tooltip renders in a closed shadow root (`src/content/index.tsx`), so page script and
  styles cannot read or restyle it; the content script runs only on `*.lightning.force.com`.
- **Residual.** A page can obscure or overlay Sextant's UI, and can detect that Sextant is installed by
  loading its web-accessible chunks (`use_dynamic_url: false`, generated by the build tool) — **O**, owner
  decision.

### T7 — A hostile import file

- **Mitigations.** Size cap before reading, prototype-polluting keys refused, per-org partitions validated
  as org ids, null-prototype containers (`src/shared/import-safety.ts` and its callers) — **T**.
- **Residual.** An import from a trusted-looking file still replaces what the user chose to replace.

### T8 — A compromised dependency

- **Mitigations.** Lockfile committed; install scripts run only for reviewed packages
  (`.npmrc` `strict-allow-scripts`, `package.json` `allowScripts`) — **E/P**. The release build records
  exactly which packages reach the bundle (12 today) with lockfile integrity, produces a CycloneDX SBOM, and
  classifies any change to bundled packages, destinations or install scripts as security-sensitive,
  requiring explicit approval (`scripts/release/release.mjs`) — **B/P**. The artifact scan catches new
  destinations, telemetry, remote code and secrets whatever their source — **B**.
- **Residual.** Malicious behaviour inside an already-approved package version, which adds no destination,
  is not detectable by these checks. Several development-only packages have known advisories
  (`AUDIT_2026-09-15.md`).

### T9 — A malicious update

- **Mitigations.** One canonical release path from a tagged commit in a clean checkout, with every gate
  above; published hashes and provenance; release classification with human approval for trust changes
  (`RELEASE_SECURITY.md`) — **B/P**. Publisher account hardening (hardware security keys, no shared
  credentials) — **P**, checklist in `RELEASE_SECURITY.md` §2.
- **Residual.** **The Chrome Web Store accepts any package from the publisher account.** Someone who holds
  that account bypasses every gate in this repository, and browsers install the update automatically. The
  controls reduce the chance of that and make it detectable afterwards; they cannot prevent it.

### T10 — Remote or string-evaluated code

- **Mitigations.** Manifest V3 forbids remote code for extension pages and workers; the CSP forbids
  `eval` — **E**, verified in the browser suite. Source guards and the artifact scan — **T/B**.
- **Residual.** None known.

### T11 — A new destination or telemetry added by accident

- **Mitigations.** T1's CSP and tests; the approved baseline of origins, permissions and CSP
  (`release-metadata/trust-baseline.json`); a trust diff on every release — **E/T/B**. Changing any of it is
  a Trust Architecture Change (`RELEASE_SECURITY.md` §4) — **P**.

### T12 — The development server exposes source or org data

- **Mitigations.** Development servers refuse the editor-launch route and deny serving environment files,
  keys and `*.gate.json` evaluation files (`scripts/dev-server-hardening.ts`) — verified with requests.
- **Residual.** Development servers still serve source to the local machine, by design.

### T13 — A write lands in the wrong org or in production by mistake

- **Mitigations.** Org resolved per request and shown in every confirmation (name, environment, host); a
  direct save to a production org or to an org Sextant cannot identify always asks and cannot be silenced
  (`src/shared/direct-deploy.ts`; `src/workspace/confirm-deploy-production.test.tsx`,
  `src/content/tooltip-confirm.test.tsx`) — **T**. Optimistic concurrency refuses to overwrite a newer value.
- **Residual.** Batches assembled in the deploy tray are sent on the tray's Deploy button after its own
  review, without the per-change dialog.

### T14 — Someone with the browser profile reads local data

- **Mitigations.** None technical, stated plainly (`SECURITY.md` §5). No credential is stored; data is
  visible and deletable per org and in full.
- **Residual.** Accepted.

### T15 — Diagnostics capture more than intended

- **Mitigations.** Off by default; bounded; credential shapes redacted; deletable; the DOM sink that exposed
  traces to the page was removed (`src/shared/diagnostics.ts`, `src/content/index.tsx`;
  `src/shared/diagnostics-privacy.test.ts`) — **T**.
- **Residual.** Traces the user turns on contain page text by design.

### T16 — The unreleased AI code ships

- **Mitigations.** Excluded at compile time (`__AI_KERNEL__`, `vite.config.ts`); the release scan fails on
  provider code; a release refuses to build with the kernel; boot deletes development AI keys in store
  builds — **B/T**.

### T17 — Client or personal data reaches the public repository or a release

- **Mitigations.** The public tree is an allow-list (`scripts/build-public-tree.mjs`); a private list of
  identifying names and public secret shapes are scanned in the tree and in every artifact; fixtures are
  synthetic by construction; credential-shaped test values are assembled at run time
  (`src/security/testing/fake-secrets.ts`) — **B/T/P**.
- **Residual.** The private repository's history contains earlier revisions; the public repository is
  planned to start without it.

### T18 — Resource exhaustion

- **Mitigations.** Explicit budgets for fan-out and storage (`src/shared/budgets.ts`,
  `storage-budget.ts`); bounded message sizes; import size caps; linear-time redaction on hostile input
  (`src/shared/secret-redaction.test.ts`) — **T**.
- **Residual.** A very large org can still take minutes to read; that is a cost, not a vulnerability.

## 5. When this model must be revisited

Before a release that adds a permission, a host, a destination, a stored category of data, a dependency
in the bundle, remote configuration, telemetry, a server or an AI provider — each a Trust Architecture
Change (`RELEASE_SECURITY.md` §4) — and after any security report that the model did not anticipate.
